# 11 AI Scenario Questions

This document compiles real-world, scenario-based interview questions and architectural approaches covering prompting decisions, RAG latency, agentic state loops, document processing, and infrastructure costs.

## Prompting, Fine-Tuning, and Safety Scenarios

### Definition & The Problem It Solves
Deciding whether to resolve a feature using prompt engineering, RAG, or fine-tuning is the most common architectural decision in AI engineering. Using the wrong approach causes high latency, bloated token bills, or incorrect outputs. This topic covers the scenarios where you must choose the appropriate model customization method, enforce safety guardrails, and prevent brand damage from model hallucinations.

When I first built an internal support helper, I attempted to fine-tune a model on our wiki articles. I quickly realized that fine-tuning did not stop the model from hallucinating details when articles changed, forcing me to pivot to a RAG architecture.

### The Real-World Analogy
I like to compare this to preparing a lawyer for a trial. Prompting is like whispering a quick instruction to them before they walk into the courtroom. Fine-tuning is like sending them to law school: they learn the professional style, format, and vocabulary of law. RAG is like handing them the specific evidence folders for the case: they read the facts directly from the files instead of trying to remember them from school.

### The Code
```python
# Architecture: Combining a guardrail check with RAG routing
class SafetyRouter:
    def __init__(self, guardrail_service, rag_engine):
        self.guardrail = guardrail_service
        self.rag = rag_engine

    def route_request(self, user_query: str) -> str:
        # Check safety policy before execution
        if not self.guardrail.is_safe(user_query):
            return "I am sorry, but I cannot answer queries violating our safety policy."
            
        # Route query to RAG if safe
        return self.rag.query(user_query)
```

### Interview Questions

#### Topic: Brand Protection and Hallucination
**Question:** You built a customer support bot using RAG. It works great in testing, but in production, users are asking questions about competitors, and the bot is hallucinating bad things about your company because it is trying to be helpful. How do you fix this architecturally?
**Expected Approach:** 
1. Implement a two-stage guardrail system. First, use an input guardrail classifier (e.g. a small, fast model or NeMo Guardrails) to identify if the user's query mentions competitors.
2. If competitor mentions are detected, route the query to a strict, pre-written prompt template: "I cannot comment on competitor pricing or features. Please visit their official page."
3. Implement output guardrail scanning (e.g. checking for company name or competitor terms in the generated response) to block any response that violates marketing policies.
4. Set the model temperature to 0.0 to ensure deterministic, safe responses.

#### Topic: RAG vs Fine-Tuning
**Question:** Your product team wants to build a feature that generates custom SQL queries based on natural language inputs from customers. They are debating whether to fine-tune a model or use standard prompting with RAG. How do you evaluate and guide this decision?
**Expected Approach:**
1. Recommend fine-tuning if the database schema is static and has a highly customized syntax that standard models struggle with. Fine-tuning is excellent for learning syntax styles and formatting rules.
2. Recommend RAG (injecting database schema, table definitions, and few-shot examples into the context window) if the database tables and columns change frequently, as fine-tuning cannot adapt to database updates in real time.
3. For a production system, use a hybrid approach: fine-tune a smaller model (like a 7B parameter model) on a dataset of 1,000+ query-to-SQL pairs to learn the custom syntax, and use RAG to inject the current active schema columns into the context window during runtime.

### References
- [Hugging Face: Fine-Tuning vs RAG Decision Guide](https://huggingface.co/blog/ray-rag)

## RAG Latency and Search Optimization Scenarios

### Definition & The Problem It Solves
RAG pipelines often suffer from high latency because they combine multiple network calls: embedding generation, vector database search, cross-encoder reranking, and LLM text generation. If user queries take more than 3 seconds to resolve, adoption drops. This topic covers the techniques to profile, optimize, and cache elements of your search pipeline to maintain high retrieval speeds.

I learned this when our RAG search latency reached 4.5 seconds. Profiling the code showed that the cross-encoder reranking model was taking 2 seconds to analyze irrelevant text chunks, prompting us to reduce the candidate list size.

### The Real-World Analogy
I like to compare optimizing RAG latency to improving service speed at a fast-food restaurant. You do not wait until a customer orders to start chopping the lettuce and baking the bread. You pre-cut the ingredients (pre-computed embeddings), keep the popular burgers warm in the tray (caching common queries), and set up separate stations for orders and pickups (decoupling retrieval from generation) so the customer gets their food in seconds.

### The Code
```python
# Latency profiling decorator to isolate bottlenecks in RAG steps
import time

def profile_step(step_name: str):
    def decorator(func):
        def wrapper(*args, **kwargs):
            start_time = time.time()
            result = func(*args, **kwargs)
            duration = time.time() - start_time
            print(f"[LATENCY PROFILE] Step '{step_name}' took {duration:.4f} seconds")
            return result
        return wrapper
    return decorator
```

### Interview Questions

#### Topic: Resolving Slow RAG Latency
**Question:** Your RAG system's end-to-end response time is 5 seconds. Users are complaining about the delay. How do you profile and optimize this pipeline to bring the latency under 1.5 seconds?
**Expected Approach:**
1. Profile the latency of each pipeline stage: embedding creation, vector search, reranking, and generation.
2. If embedding creation is slow, switch from a cloud API to a local, GPU-hosted model (like `all-MiniLM-L6-v2`) to eliminate network hops.
3. If vector search is slow, build an HNSW index on the database (e.g. configuring `m = 16` and `ef_construction = 64` in pgvector) to avoid linear scans.
4. If reranking is the bottleneck, reduce the candidate list sent to the cross-encoder (e.g. from 50 candidates to 15) or switch to a faster reranking model.
5. If generation is slow, enable token streaming (Server-Sent Events) in the backend so the user sees the output instantly as it is generated, improving the perceived latency.

#### Topic: Search Cache Architecture
**Question:** Your application receives 10,000 queries per hour, and many of them are repetitive. How do you implement a caching strategy for your RAG system to save costs and reduce latency?
**Expected Approach:**
1. Implement a two-layer cache: an exact cache and a semantic cache.
2. For exact matches, query a Redis key-value store using a hash of the user's normalized query string. If cached, return the answer instantly (0.5ms latency).
3. For semantic matches, use a semantic cache: embed the incoming query, search a vector database containing previous queries, and if a previous query matches with a cosine similarity score of > 0.96, return the cached answer.
4. Configure an expiration policy (TTL) on cache records (e.g. 24 hours) to prevent the system from returning outdated business facts.

### References
- [Pinecone: How to optimize vector database queries](https://www.pinecone.io/learn/vector-search-performance/)

## Agentic Workflows and State Loops Scenarios

### Definition & The Problem It Solves
Agentic workflows use model loops to solve multi-step tasks. In production, these systems can get stuck: an agent can get stuck in an infinite loop where it queries a tool, receives an error, rephrases the query, receives the same error, and repeats the cycle indefinitely. This topic covers the techniques to build state machines with strict guards, timeouts, and state tracking to prevent runaways.

When I first built a search agent, it got stuck in a loop when it looked for a page that did not exist. The agent spent ten minutes querying different search phrases, running up our API bill before we terminated the process manually.

### The Real-World Analogy
The easiest way I understand this is to compare the agent to a robotic vacuum cleaner. If you do not install collision sensors and a timeout timer (state guards), the vacuum can get stuck behind a chair leg, bumping into it repeatedly and draining its battery until you physically pick it up and move it.

### The Code
```python
# State loop guard tracking attempts to prevent runaway execution
class AgentStateGuard:
    def __init__(self, max_loops: int = 5):
        self.max_loops = max_loops

    def check_loop_safety(self, current_state: dict) -> bool:
        # Check if current execution loop index exceeds the safety limit
        loop_count = current_state.get("loop_count", 0)
        if loop_count >= self.max_loops:
            # Trigger fallback transition
            return False
        return True
```

### Interview Questions

#### Topic: Debugging Infinite Loops in LangGraph Agents
**Question:** You deployed a customer research agent built with LangGraph. In production, some user queries trigger infinite loops where the agent repeatedly calls the same search tool with slight variations without ever returning a final answer. How do you debug and resolve this?
**Expected Approach:**
1. Add a `loop_counter` field to the shared LangGraph state object.
2. In the node execution functions, increment the `loop_counter` by 1 every time the search node is entered.
3. Modify the conditional edges: if `loop_counter` exceeds a maximum limit (e.g. 5 iterations), force a route transition to a fallback node instead of the search node.
4. The fallback node should summarize the research gathered so far and output: "I searched for the details but reached my search limit. Here is what I found..."
5. Connect Langfuse or LangSmith to trace the execution steps, identifying which query structures trigger these loops.

#### Topic: Tool Execution Failures
**Question:** Your agent needs to call a third-party weather API. The API is rate-limited and regularly returns 429 status codes. How do you write the agent's tool execution code to handle these failures gracefully?
**Expected Approach:**
1. Implement exponential backoff with jitter directly inside the tool function execution block.
2. If the API returns a 429 after retries, return a structured error message to the agent: "Error: Weather API is currently rate-limited. You must inform the user and suggest they try again later."
3. Do not raise a raw Python exception that crashes the server. Instead, return the error message as a string tool result so the agent can parse it and reply to the user.
4. Register the error event in your tracing log to track tool health metrics.

### References
- [LangGraph: How to add validation and loop protection](https://langchain-ai.github.io/langgraph/how-tos/input-output-schema/)

## Large Document Processing and Context Window Scenarios

### Definition & The Problem It Solves
Processing large documents (like 500-page manuals or financial filings) exceeds the context window limits of most models, or degrades generation quality because of the "lost in the middle" effect. This topic covers the chunking, metadata tagging, Map-Reduce patterns, and context pruning techniques needed to summarize and query large documents without crashing your application.

When I built a PDF analyzer, upload requests for large files régulièrement crashed our server due to out-of-memory errors. Implementing a streaming page-by-page parser resolved the memory issues.

### The Real-World Analogy
I like to compare processing a massive document to reading a 1000-page historical textbook to answer a specific question. You do not try to memorize the whole book in one reading (context limit). You check the index at the back (metadata), go directly to the relevant chapters, read the summaries of those chapters (Map-Reduce), and compile your answer from those pages.

### The Code
```python
# Batch processing pages to avoid loading full files in memory
class StreamingDocumentParser:
    def process_large_document_in_batches(self, file_path: str, batch_size: int = 10):
        # Open file stream and read pages sequentially
        with open(file_path, "r") as f:
            batch = []
            for line in f:
                batch.append(line)
                if len(batch) >= batch_size:
                    yield "".join(batch)
                    batch = []
            if batch:
                yield "".join(batch)
```

### Interview Questions

#### Topic: Processing a 500-Page PDF without blowing up context limits
**Question:** A client wants to upload a 500-page PDF report and ask global questions (e.g. "What are the key financial risks across all departments?"). How do you design an ingestion and query system that handles this file size without blowing up the model's context window?
**Expected Approach:**
1. Do not pass the entire PDF text to the model. Implement a Map-Reduce pipeline.
2. During ingestion, split the document into smaller chapters or page ranges (e.g. 5 pages per chunk) and parse them using a structured tool like LlamaParse.
3. For each chunk, run a fast summarization step to extract key financial points, saving these summaries next to the source text.
4. When the user asks a global question, use a two-step query process: first, query the index of summaries to identify which sections of the report mention financial risks.
5. Retrieve the specific raw chunks for those sections, pass them to the model, and synthesize the final answer.

#### Topic: High-Accuracy Extraction from Scanned Documents
**Question:** Your database contains thousands of scanned receipts and invoices stored as low-resolution images. How do you design an extraction pipeline to retrieve total values and dates with near 100% accuracy?
**Expected Approach:**
1. Use a Vision Language Model (VLM) like Claude 3.5 Sonnet to process the receipt images directly, as standard text OCR loses formatting.
2. Prompt the model to return data in a structured JSON schema using Pydantic, enforcing fields for `total_amount` and `transaction_date`.
3. Wrap the model API call with the `instructor` library, setting `max_retries = 3` to automatically trigger self-correction loops if the model outputs invalid formats or missing fields.
4. Write custom Pydantic validators to verify that the numbers add up correctly: e.g. checking that the sum of line items matches the `total_amount` value.

### References
- [LlamaIndex: Ingesting and querying large documents](https://docs.llamaindex.ai/en/stable/module_guides/loading/connector/llama_parse/)

## Infrastructure and Cost Optimization Scenarios

### Definition & The Problem It Solves
Production AI systems face high operational costs: cloud GPU instances are expensive, and calling commercial model APIs scales linearly with token counts. If left unoptimized, high-traffic applications run up large bills that wipe out profit margins. This topic covers the techniques to optimize cost: choosing appropriate models, using quantization formats, deploying high-throughput serving engines, and routing requests through cost-aware API gateways.

I learned this when our OpenAI API bill reached $2,000 in a week during user testing. Migrating our simple text classification steps to a local, quantized 3B parameter model running on a cheap cloud instance cut our monthly bills by 85%.

### The Real-World Analogy
I like to compare cost optimization to managing household electricity costs. You do not leave your high-power heaters running at max power in every room all day (unoptimized 70B cloud models). You use small heaters in occupied rooms (small language models), insulate the walls to retain heat (caching prompts), and install smart thermostats that adjust temperature based on occupancy (gateway routing rules).

### The Code
```python
# Cost calculation helper mapping input and output token rates
class TokenCostCalculator:
    def __init__(self):
        # Cost per 1 million tokens (values are mock examples)
        self.pricing = {
            "gpt-4o": {"input": 5.0, "output": 15.0},
            "gpt-4o-mini": {"input": 0.15, "output": 0.60}
        }

    def compute_cost(self, model: str, input_tokens: int, output_tokens: int) -> float:
        model_rates = self.pricing.get(model, {"input": 0.0, "output": 0.0})
        input_cost = (input_tokens / 1_000_000) * model_rates["input"]
        output_cost = (output_tokens / 1_000_000) * model_rates["output"]
        return input_cost + output_cost
```

### Interview Questions

#### Topic: API Rate Limits and Failovers
**Question:** Your application relies on OpenAI APIs. During peak hours, you regularly hit rate limit errors (429), causing user requests to fail. How do you design a resilient architecture to resolve this?
**Expected Approach:**
1. Set up an API gateway like LiteLLM to sit between your backend application and the model providers.
2. Configure a load-balanced pool of API keys across multiple accounts to distribute the token limits.
3. Set up automatic fallback routing rules in the gateway: if the primary model (e.g. OpenAI `gpt-4o`) returns a 429 error, route the request to a fallback provider (e.g. Anthropic `claude-3-5-sonnet` or a self-hosted vLLM engine).
4. Implement a token bucket rate limiter in your backend to throttle high-volume users before they exhaust your API limits.

#### Topic: Cost Optimization Strategies
**Question:** Your company is deploying an application that processes 100,000 customer emails daily to classify sentiment and route them to departments. Using a large model (gpt-4o) costs too much at this volume. How do you design this system to be cost-effective?
**Expected Approach:**
1. Do not use a large model for simple classification tasks.
2. Fine-tune a small, open-source model (like Llama 3.2 3B) specifically on a dataset of 5,000 labeled emails.
3. Host the fine-tuned 3B model locally or on a cheap server using a high-throughput engine like vLLM.
4. Compressing the model weights to 4-bit (quantization) allows it to run on a single, cheap GPU.
5. If the model encounters a highly ambiguous email where its classification confidence is low (e.g. < 70%), route only that specific query to a larger model (gpt-4o) as a fallback.

### References
- [LiteLLM: Load Balancing and Failover](https://docs.litellm.ai/docs/proxy/load_balancing)
- [vLLM: Engine Performance Tips](https://docs.vllm.ai/en/latest/models/engine_args.html)
