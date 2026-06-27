# 01 Foundations: Transformers and RLM

This guide covers NLP foundations: transition to Transformers, tokenization, embeddings, vector math, the attention mechanism, autoregressive generation, Recursive Language Models (RLMs), and probability sampling.

## The Transition from LSTMs to Transformers

### Definition & The Problem It Solves
Recurrent Neural Networks (RNNs) and Long Short-Term Memory networks (LSTMs) processed text sequentially, word-by-word. This sequential bottleneck meant that training could not be parallelized across GPUs, and models struggled to maintain long-range dependencies (forgetting earlier words in long paragraphs). The Transformer architecture, introduced in 2017, solved this by processing the entire input sequence simultaneously using self-attention, enabling parallel training on massive datasets and catching connections between words regardless of their distance.

When I first worked on text summarization with LSTMs, my models regularly hallucinated because they forgot the beginning of a 500-word article by the time they reached the end. Moving to Transformers eliminated this constraint, making long-context processing practical.

### The Real-World Analogy
I like to compare LSTMs to reading a book through a keyhole: you can only see one word at a time, and you must hold the context of the sentence in your memory as you move the page. A Transformer is like opening the door and looking at the entire page at once: you see all the words simultaneously, letting your eyes jump back and forth to connect pronouns to their nouns.

### The Code
```python
# Conceptual comparison of sequential processing vs parallel attention mapping
import numpy as np

class SequentialProcessingMock:
    def process_sequentially(self, text: list[str]) -> list[np.ndarray]:
        # Sequentially updates hidden state, simulating LSTM behavior
        hidden_state = np.zeros(128)
        states = []
        for word in text:
            # Each step depends strictly on the previous hidden_state (sequential bottleneck)
            hidden_state = np.tanh(hidden_state + np.random.randn(128) * hash(word) % 1)
            states.append(hidden_state)
        return states

class ParallelTransformerMock:
    def process_parallel(self, text: list[str]) -> np.ndarray:
        # Simulate simultaneous context processing via an attention matrix
        seq_len = len(text)
        # Create attention relationships between all tokens at once
        attention_matrix = np.random.rand(seq_len, seq_len)
        # Normalize the matrix rows
        attention_matrix = attention_matrix / attention_matrix.sum(axis=-1, keepdims=True)
        return attention_matrix
```

### Interview Questions
- Why do LSTMs suffer from the vanishing gradient problem when processing long sequences?
- Explain how self-attention enables parallel computation during training.
- What are positional encodings and why do Transformers need them?

### References
- [Attention Is All You Need (Vaswani et al., 2017)](https://arxiv.org/abs/1706.03762)

## Tokens, Context Windows, and Context Constraints

### Definition & The Problem It Solves
Models do not read text directly; they process numbers called Tokens. Tokenizers split text into sub-word pieces. The Context Window is the maximum number of tokens a model can process in a single request/response cycle. Managing this window is critical: every token incurs financial cost, increases processing latency, and exceeding the limit causes the model to truncate historical context or crash the API call.

I learned this the hard way when I built a support bot. I passed the entire customer conversation history to the model on every message. As the conversation grew, the bot's API calls began failing because the token count exceeded the model limits, which forced me to implement sliding-window message eviction.

### The Real-World Analogy
The easiest way I understand this is to compare the context window to a whiteboard in a meeting room. The whiteboard has limited space (the context window). You write down facts (tokens) as the meeting progresses. If the board fills up, you cannot write new ideas unless you erase older notes. If you erase the wrong notes, the team loses track of what was agreed on at the start of the meeting.

### The Code
```python
# Checking and trimming token lengths using tiktoken to prevent API context overflows
import tiktoken

class ContextWindowManager:
    def __init__(self, model_name: str = "gpt-4", max_allowed_tokens: int = 4096):
        self.encoder = tiktoken.encoding_for_model(model_name)
        self.max_allowed_tokens = max_allowed_tokens

    def count_tokens(self, text: str) -> int:
        return len(self.encoder.encode(text))

    def trim_history(self, messages: list[dict], reserve_tokens: int = 500) -> list[dict]:
        trimmed_messages = []
        current_token_count = 0
        limit = self.max_allowed_tokens - reserve_tokens

        # Process messages in reverse (most recent first) to keep recent context
        for msg in reversed(messages):
            msg_tokens = self.count_tokens(msg["content"])
            if current_token_count + msg_tokens > limit:
                break # Stop adding older messages to avoid context blowup
            trimmed_messages.insert(0, msg)
            current_token_count += msg_tokens
            
        return trimmed_messages
```

### Interview Questions
- Why do models calculate costs per token instead of per character or word?
- Describe the difference between Byte-Pair Encoding (BPE) and WordPiece tokenization.
- How do you handle a system query when the retrieved database context exceeds the context window?

### References
- [Hugging Face: Tokenizers guide](https://huggingface.co/docs/tokenizers/index)
- [OpenAI: Tokenizer Tool](https://platform.openai.com/tokenizer)

## Text Embeddings and Vector Space

### Definition & The Problem It Solves
Text Embeddings convert natural language into arrays of numbers (vectors) representing coordinates in a high-dimensional vector space. These coordinate points capture semantic meaning: words or sentences with similar meanings are plotted close together. This mathematical mapping solves the problem of semantic search, allowing computer programs to evaluate similarity without relying on exact keyword matches.

When I first built a search feature, I used SQL LIKE queries. If a user searched for "feline", they missed articles containing "cat". Converting text to embeddings resolved this, as the vectors for "cat" and "feline" map to coordinates in the same vector neighborhood.

### The Real-World Analogy
I like to compare this to plotting movies on a graph. Imagine a graph where the horizontal axis is "Action" and the vertical axis is "Romance". You plot movies: an action movie maps to (10, 1), and a romantic comedy maps to (1, 10). If a user asks for "something with fighting", you check coordinates close to the action axis. Embeddings behave similarly, but instead of 2 axes, they use 1,536 or more dimensions to capture complex grammar and themes.

### The Code
```python
# Generating mock embedding vectors to compare text similarities
import numpy as np

class MockEmbeddingEngine:
    def __init__(self, dimension: int = 1536):
        self.dimension = dimension

    def get_embedding(self, text: str) -> list[float]:
        # Simulates generating a unit-normalized vector for a text string
        np.random.seed(abs(hash(text)) % (2**32 - 1))
        raw_vector = np.random.randn(self.dimension)
        # Normalize to unit length (L2 norm)
        normalized_vector = raw_vector / np.linalg.norm(raw_vector)
        return normalized_vector.tolist()
```

### Interview Questions
- What does it mean when we say an embedding vector is L2-normalized?
- How does the dimensionality of an embedding model affect latency and storage costs?
- Why do embeddings sometimes fail to capture negation (e.g. "not good" mapping close to "good")?

### References
- [OpenAI: Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)
- [Cohere: What are Word Embeddings?](https://cohere.com/blog/sentence-word-embeddings)

## Vector Math for Developers (Cosine Similarity and Dot Product)

### Definition & The Problem It Solves
Vector Math evaluates the similarity of two text embeddings. The two primary metrics are **Dot Product** (measures matching direction and magnitude) and **Cosine Similarity** (measures only matching angle direction, ignoring length magnitude). This math solves the search retrieval bottleneck: instead of querying database records sequentially, vector operations calculate distance scores across millions of documents in milliseconds.

When I first computed document matches, I used raw nested loops, which froze my server. I resolved this by standardizing my vectors to unit length, which allows Cosine Similarity to be calculated via simple, hardware-optimized Dot Product operations.

### The Real-World Analogy
The easiest way I understand this is to visualize two arrows shot from the center of a target. Cosine similarity evaluates the angle between the two arrows: if they point in the exact same direction, the score is 1.0 (perfect match), regardless of how far they flew. The dot product is like measuring how much they landed in the same zone while accounting for their length.

### The Code
```python
# Calculating similarity using Dot Product and Cosine Similarity in NumPy
import numpy as np

class VectorMathUtils:
    @staticmethod
    def dot_product(v1: np.ndarray, v2: np.ndarray) -> float:
        # Measures the sum of the products of corresponding coordinates
        return float(np.dot(v1, v2))

    @staticmethod
    def cosine_similarity(v1: np.ndarray, v2: np.ndarray) -> float:
        # Divides dot product by the product of vector magnitudes
        numerator = np.dot(v1, v2)
        denominator = np.linalg.norm(v1) * np.linalg.norm(v2)
        if denominator == 0:
            return 0.0
        return float(numerator / denominator)

# Example demonstration
if __name__ == "__main__":
    v_query = np.array([1.0, 0.0, 0.0])
    v_doc1 = np.array([0.9, 0.1, 0.0]) # Close angle
    v_doc2 = np.array([0.0, 1.0, 0.0]) # Orthogonal angle
    
    print(f"Cosine Similarity (Query, Doc1): {VectorMathUtils.cosine_similarity(v_query, v_doc1):.4f}")
    print(f"Cosine Similarity (Query, Doc2): {VectorMathUtils.cosine_similarity(v_query, v_doc2):.4f}")
```

### Interview Questions
- Why are Cosine Similarity and Dot Product mathematically identical if vectors are L2-normalized?
- When should you use Euclidean distance over Cosine Similarity for vector lookups?
- How do you optimize similarity math operations using matrix multiplications?

### References
- [Pinecone: Cosine Similarity Guide](https://www.pinecone.io/learn/vector-similarity/)

## The Transformer Core (Query, Key, Value Mechanisms)

### Definition & The Problem It Solves
The Query-Key-Value (QKV) mechanism is the core of the Self-Attention block. It calculates weights between tokens dynamically: for any word, it determines how much attention to pay to other words in the sentence. This solves the limitation of static word vectors (like Word2Vec), allowing the word "bank" to change its vector state depending on whether it is next to "river" or "money".

When I first learned how attention works, I struggled to understand why we needed three separate vectors (Q, K, and V) for a single word. I realized that a word needs different representations depending on whether it is asking a question, matching a category, or carrying the actual content.

### The Real-World Analogy
I like to compare this to searching for a video on YouTube. The search term you type in the search box is the **Query (Q)**. The titles of the videos listed on the YouTube server are the **Keys (K)**. The actual video content you watch is the **Value (V)**. The system calculates the similarity between your query and each video title (Q dot product K), and uses this correlation score to deliver the correct video contents (Values) to your screen.

### The Code
```python
# Simple mathematical representation of scaled dot-product attention
import numpy as np

class ScaledDotProductAttention:
    def calculate_attention(self, Q: np.ndarray, K: np.ndarray, V: np.ndarray) -> np.ndarray:
        # Scale factor is square root of Key dimension (dk) to prevent vanishing gradients
        dk = K.shape[-1]
        
        # 1. Compute similarity logits (Q * K^T)
        scores = np.matmul(Q, K.T) / np.sqrt(dk)
        
        # 2. Convert raw scores to probability distribution via Softmax
        exp_scores = np.exp(scores - np.max(scores, axis=-1, keepdims=True)) # numeric stability
        attention_weights = exp_scores / np.sum(exp_scores, axis=-1, keepdims=True)
        
        # 3. Multiply weights by Values
        output = np.matmul(attention_weights, V)
        return output
```

### Interview Questions
- Explain the role of the scale factor (1/sqrt(dk)) in Scaled Dot-Product Attention.
- What is Multi-Head Attention and why is it preferred over Single-Head Attention?
- How do you construct a Masked Attention layer for decoder models?

### References
- [Illustrated Guide to Transformers Attention](https://jalammar.github.io/illustrated-transformer/)

## Autoregressive Generation

### Definition & The Problem It Solves
Autoregressive Generation is the technique where a model generates text sequentially, one token at a time. The model predicts the next token based on all previous tokens, appends the newly generated token to its input context, and feeds the updated sequence back in to predict the subsequent token. This solves the challenge of generating coherent multi-sentence text.

I realized how critical this was when I observed API latency metrics. Because generation is autoregressive, returning 100 tokens requires running the model forward 100 separate times, explaining why response latency scales linearly with output length.

### The Real-World Analogy
I like to compare this to writing on a typewriter. You type one letter. To decide the next letter, your eyes inspect everything currently printed on the page. You stamp the next key down, and slide your eyes to include it, repeating the loop. You cannot write the last paragraph of the letter without typing the intermediate sentences first.

### The Code
```python
# A conceptual autoregressive next-token generation loop
import numpy as np

class AutoregressiveGenerator:
    def __init__(self, vocab: list[str]):
        self.vocab = vocab

    def predict_next_token_probs(self, context_tokens: list[str]) -> np.ndarray:
        # Simulate model weights outputting probability scores for the next token
        np.random.seed(abs(hash("".join(context_tokens))) % 2**32)
        raw_logits = np.random.randn(len(self.vocab))
        # Softmax normalization
        probs = np.exp(raw_logits) / np.sum(np.exp(raw_logits))
        return probs

    def generate(self, prompt: str, max_tokens: int = 5) -> str:
        tokens = prompt.split()
        for _ in range(max_tokens):
            probs = self.predict_next_token_probs(tokens)
            # Sample next token index based on probability distribution
            next_idx = np.random.choice(len(self.vocab), p=probs)
            next_token = self.vocab[next_idx]
            tokens.append(next_token)
            if next_token == "[EOS]": # Stop if End of Sequence token is hit
                break
        return " ".join(tokens)
```

### Interview Questions
- Why does autoregressive generation cause high inference GPU memory usage (e.g. KV Cache)?
- How does speculative decoding speed up autoregressive model inference?
- What are the differences between greedy decoding and nucleus sampling during generation?

### References
- [Hugging Face: Generation Strategies](https://huggingface.co/docs/transformers/generation_strategies)

## Recursive Language Models (RLM)

### Definition & The Problem It Solves
Recursive Language Models (RLMs) break down complex, multi-step queries by recursively decomposing them into nested sub-tasks, executing each sub-task, and feeding the child outputs back into the parent context. Traditional LLMs process text in a single left-to-right path, which causes them to fail at long planning problems because they attempt to write the final answer before outlining intermediate steps. RLMs solve this by structuring generation as a tree execution loop.

When I designed a code migration bot, passing the entire repository to an LLM resulted in context overload and broken code. Implementing a recursive decomposition pattern where the model analyzed modules, resolved their components, and rolled the results up resolved the context limits.

### The Real-World Analogy
The easiest way I understand this is to compare it to a project manager distributing work in a company. Instead of the manager trying to write a 50-page technical specifications document alone (standard LLM), they break the document into 5 modules, assign each module to a developer, wait for the developers to complete their code sections, and recursively assemble the results into the final report.

### The Code
```python
# Conceptual loop of a Recursive Language Model resolving tasks
import json

class RecursiveTaskResolver:
    def __init__(self, mock_llm_callback):
        self.llm = mock_llm_callback

    def resolve(self, task_description: str, depth: int = 0, max_depth: int = 3) -> dict:
        # Prevent infinite recursion loops
        if depth >= max_depth:
            return {"task": task_description, "status": "failed", "error": "Max depth exceeded"}

        # Step 1: LLM analyzes task and decides if it needs decomposition
        planning_prompt = f"Analyze task: '{task_description}'. Can it be solved directly, or do you need to split it into subtasks? Respond in JSON format: {{'can_solve': bool, 'subtasks': list}}"
        plan = json.loads(self.llm(planning_prompt))

        if plan["can_solve"]:
            # Base case: Solve task directly
            execution_prompt = f"Execute direct task: '{task_description}'"
            result = self.llm(execution_prompt)
            return {"task": task_description, "status": "resolved", "result": result}
        
        # Recursive step: Resolve subtasks and roll up results
        subtask_results = []
        for subtask in plan["subtasks"]:
            subtask_res = self.resolve(subtask, depth + 1, max_depth)
            subtask_results.append(subtask_res)

        # Synthesize final answer from subtask results
        synthesis_prompt = f"Synthesize main task '{task_description}' using results: {json.dumps(subtask_results)}"
        final_result = self.llm(synthesis_prompt)
        return {"task": task_description, "status": "resolved", "result": final_result}
```

### Interview Questions
- How do Recursive Language Models prevent context overflow in long planning tasks?
- What are the debugging and monitoring challenges of recursive LLM loops?
- How does recursive decomposition differ from standard chain-of-thought prompting?

### References
- [Stanford NLP: Recursive Deep Learning](https://nlp.stanford.edu/sentiment/rnn.html)
- [Microsoft Research: Hierarchical planning in Agentic structures](https://www.microsoft.com/en-us/research/blog/hierarchical-planning-with-large-language-models/)

## Model Parameters (Temperature, Top-P, Top-K)

### Definition & The Problem It Solves
Temperature, Top-P, and Top-K control next-token probability selection. **Temperature** scales raw logits before Softmax (lower temperature peaks the distribution making it deterministic, higher temperature flattens it making it diverse). **Top-K** limits selection to the K highest probability tokens. **Top-P** (nucleus sampling) dynamically limits selection to a subset of tokens whose cumulative probability exceeds P.

I learned this parameters balance when configuring a code generator versus a creative writing bot. I set the code generator temperature to 0.0 to guarantee syntax predictability, and set the creative writer temperature to 0.8 with Top-P at 0.9 to prevent repetitive phrases.

### The Real-World Analogy
Imagine a waiter presenting a tray of dessert choices. Temperature at 0.0 is telling the waiter: "Bring me the single most popular cake". High temperature is flattening the selection: you consider any dessert on the menu. Top-K at 3 is telling the waiter: "Only show me the top 3 best-selling cakes". Top-P at 0.8 is telling them: "Gather the best desserts until their cumulative popularity score covers 80% of sales, and let me pick one from that group".

### The Code
```python
# Simulating temperature scaling, Top-K, and Top-P filtering on logits
import numpy as np

class ProbabilitySampler:
    @staticmethod
    def sample_next_token(logits: np.ndarray, temperature: float = 1.0, top_k: int = 0, top_p: float = 1.0) -> int:
        # 1. Apply temperature scaling
        if temperature > 0:
            logits = logits / temperature
        else:
            # Deterministic greedy pick
            return int(np.argmax(logits))

        # 2. Apply Top-K filtering
        if top_k > 0:
            # Find the threshold value of the kth element
            indices_to_remove = logits < np.partition(logits, -top_k)[-top_k]
            logits[indices_to_remove] = -float("Inf") # Eliminate from selection

        # Convert logits to probabilities
        exp_logits = np.exp(logits - np.max(logits)) # stability
        probs = exp_logits / np.sum(exp_logits)

        # 3. Apply Top-P (Nucleus) filtering
        if top_p < 1.0:
            sorted_indices = np.argsort(probs)[::-1]
            cumulative_probs = np.cumsum(probs[sorted_indices])
            
            # Identify indices to remove (cumulative prob exceeds threshold)
            sorted_indices_to_remove = cumulative_probs > top_p
            # Shift indices to keep the first token that exceeds the threshold
            sorted_indices_to_remove[1:] = sorted_indices_to_remove[:-1].copy()
            sorted_indices_to_remove[0] = False
            
            indices_to_remove = sorted_indices[sorted_indices_to_remove]
            probs[indices_to_remove] = 0.0
            # Recalculate normalized probabilities
            probs = probs / np.sum(probs)

        # Sample from the final probability distribution
        return int(np.random.choice(len(probs), p=probs))
```

### Interview Questions
- Why does setting Temperature to 0.0 bypass Top-P and Top-K filter calculations?
- How does high temperature value cause gibberish generation at token levels?
- What are the latency impacts of Top-P sorting calculations in high-throughput inference engines?

### References
- [Hugging Face: Nucleus Sampling](https://huggingface.co/blog/how-to-generate)
- [arXiv: The Curious Case of Neural Text Degeneration (Holtzman et al.)](https://arxiv.org/abs/1904.09751)
