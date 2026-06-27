# 04 Advanced RAG Architecture

This guide covers advanced Retrieval-Augmented Generation architectures: naive vs advanced pipelines, query transformation (HyDE), diversity algorithms (MMR), cross-encoder reranking, and self-correcting agentic search.

## Naive RAG vs Advanced RAG Pipelines

### Definition & The Problem It Solves
Naive RAG is a basic search pipeline: it takes a user query, generates an embedding, searches a vector database, puts the top matches into a prompt, and calls the LLM. In production, Naive RAG fails because user queries are often vague, chunk retrieval grabs irrelevant data, and important details get lost. Advanced RAG solves this by introducing pre-retrieval validation (query transformation), retrieval optimization, and post-retrieval processing (reranking) to ensure the model only receives highly relevant context.

When I deployed my first Naive RAG customer bot, it regularly failed because users typed conversational queries like "Can you help me find my previous bill?". The system embedded the word "help" and returned basic tutorial articles instead of billing records.

### The Real-World Analogy
I like to compare Naive RAG to a library assistant who runs to the bookshelf the second you speak. If you ask: "Where is that book about the war?", they run off and bring back the first five books containing the word "war" in the title. Advanced RAG is like an experienced research librarian: they ask you to clarify which war, search the index using synonyms, retrieve fifty candidate articles, scan their tables of contents, and hand you the top three most relevant pages.

### The Code
```python
# Comparison flow: Naive RAG vs Advanced RAG structure
class NaiveRAGPipeline:
    def __init__(self, db, llm):
        self.db = db
        self.llm = llm

    def execute(self, query: str) -> str:
        # Simple, fragile retrieval
        chunks = self.db.similarity_search(query, k=3)
        context = "\n".join(chunks)
        return self.llm.generate(prompt=query, context=context)

class AdvancedRAGPipeline:
    def __init__(self, query_transformer, retriever, reranker, llm):
        self.transformer = query_transformer
        self.retriever = retriever
        self.reranker = reranker
        self.llm = llm

    def execute(self, query: str) -> str:
        # 1. Transform user query (Pre-retrieval)
        refined_query = self.transformer.transform(query)
        # 2. Retrieve broad candidate list (Retrieval)
        candidates = self.retriever.retrieve(refined_query, k=20)
        # 3. Filter candidates using heavy model (Post-retrieval)
        best_chunks = self.reranker.rerank(refined_query, candidates, top_n=5)
        # 4. Generate final answer
        context = "\n".join(best_chunks)
        return self.llm.generate(prompt=refined_query, context=context)
```

### Interview Questions
- Why does Naive RAG suffer from the "lost in the middle" context window problem?
- What are the major pipeline stages that differentiate Advanced RAG from Naive RAG?
- How do you measure the latency overhead introduced by each stage of an Advanced RAG pipeline?

### References
- [Retrieval-Augmented Generation for Large Language Models: A Survey (Gao et al., 2023)](https://arxiv.org/abs/2312.10997)

## Query Transformation and Hypothetical Document Embeddings (HyDE)

### Definition & The Problem It Solves
Query Transformation modifies raw user input to improve retrieval. A key method is **Hypothetical Document Embeddings (HyDE)**. Users write short, question-style queries, whereas documents are written as long, statement-style paragraphs. This mismatch in style makes search less accurate. HyDE solves this by using a fast LLM to generate a fake ("hypothetical") answer to the user's query, embedding that fake answer, and using it to search the vector database. This matches statement-style vectors to other statement-style vectors, improving search accuracy.

I learned this when users searched: "How do I update my profile picture?". Searching the database for that exact question returned few matches. Using HyDE to first generate a fake instruction step, and then searching with that embedding, retrieved the actual database guide.

### The Real-World Analogy
The easiest way I understand this is to compare search to matching a key to a lock. A user's query is a rough description of the key. Instead of trying to open the locks with a description, you use a 3D printer (the LLM) to print a cheap plastic model of the key based on that description. Even if the plastic key is not perfect, its shape matches the actual lock much better than a written description would.

### The Code
```python
# Implementing the HyDE query transformation pipeline step
class HyDEGenerator:
    def __init__(self, llm_client, vector_db):
        self.llm = llm_client
        self.db = vector_db

    def get_hypothetical_embedding(self, query: str) -> list[float]:
        # Generate a hypothetical answer first
        generation_prompt = f"Write a paragraph explaining the answer to this question: {query}"
        hypothetical_doc = self.llm.call(prompt=generation_prompt)
        
        # Embed the generated hypothetical document
        embedding = self.db.embed(hypothetical_doc)
        return embedding

    def search(self, query: str) -> list[str]:
        hypothetical_vector = self.get_hypothetical_embedding(query)
        # Search the database using the hypothetical embedding
        return self.db.search_by_vector(hypothetical_vector, k=5)
```

### Interview Questions
- What are the risks of using HyDE if the LLM generates a highly hallucinated hypothetical document?
- When should you use query translation/expansion instead of HyDE?
- How does query rewriting help resolve pronoun references in multi-turn conversations?

### References
- [Precise Zero-Shot Dense Retrieval without Relevance Labels (Gao et al., 2022)](https://arxiv.org/abs/2212.10496)

## Retrieval Diversity via Max Marginal Relevance (MMR)

### Definition & The Problem It Solves
Max Marginal Relevance (MMR) is a search ranking algorithm that balances relevance with diversity. When you retrieve the top 5 chunks using standard similarity search, you often get 5 paragraphs that say the exact same thing because they are close to each other in vector space. This wastes context window space and increases costs. MMR solves this by selecting candidate chunks sequentially: it penalizes chunks that are too similar to already selected chunks, forcing the search results to cover different aspects of the topic.

When I built a search tool for product reviews, standard search returned five identical complaints about delivery delays. Implementing MMR allowed the system to retrieve one complaint about delivery, one about product quality, and one about size fit.

### The Real-World Analogy
I like to compare search to hiring team members. If you need five workers, you do not hire five people who went to the same school and have the same skills (standard search). You hire a lead engineer first, and then look for a designer, a marketer, and an accountant. You choose team members who bring new skills (diversity) rather than repeating what you already have.

### The Code
```python
# Re-ranking candidates using a simplified Max Marginal Relevance algorithm
import numpy as np

class MMRSelector:
    def select_mmr(self, query_vector: np.ndarray, candidate_vectors: list[np.ndarray], 
                   lambda_param: float = 0.5, top_n: int = 3) -> list[int]:
        # lambda_param: 1.0 is pure relevance, 0.0 is pure diversity
        selected_indices = []
        unselected_indices = list(range(len(candidate_vectors)))
        
        # Helper function to compute cosine similarity
        def sim(v1, v2):
            return np.dot(v1, v2) / (np.linalg.norm(v1) * np.linalg.norm(v2))

        # Select the single most relevant chunk first
        similarities = [sim(query_vector, cv) for cv in candidate_vectors]
        first_pick = int(np.argmax(similarities))
        selected_indices.append(first_pick)
        unselected_indices.remove(first_pick)

        while len(selected_indices) < top_n and len(unselected_indices) > 0:
            best_mmr_score = -float("inf")
            idx_to_pick = -1
            
            for idx in unselected_indices:
                relevance = sim(query_vector, candidate_vectors[idx])
                # Calculate maximum similarity with already selected elements
                redundancy = max([sim(candidate_vectors[idx], candidate_vectors[s]) for s in selected_indices])
                
                # MMR formula: lambda * relevance - (1 - lambda) * redundancy
                mmr_score = lambda_param * relevance - (1.0 - lambda_param) * redundancy
                if mmr_score > best_mmr_score:
                    best_mmr_score = mmr_score
                    idx_to_pick = idx
            
            selected_indices.append(idx_to_pick)
            unselected_indices.remove(idx_to_pick)

        return selected_indices
```

### Interview Questions
- How does the lambda parameter in the MMR formula control the balance between diversity and relevance?
- Why is MMR computationally more expensive than standard cosine similarity sorting?
- How does retrieval diversity protect against model hallucination?

### References
- [The Use of MMR in Document Retrieval and Summarization (Carbonell & Goldstein, 1998)](https://dl.acm.org/doi/10.1145/290941.291025)

## Two-Stage Retrieval and Reranking

### Definition & The Problem It Solves
Two-Stage Retrieval splits search into two phases. The first phase uses a fast, computationally cheap model (like a vector database query or BM25 keyword search) to retrieve a broad list of candidates (e.g. 50 chunks). The second phase uses a heavy, highly accurate model (a **Cross-Encoder Reranker**) to score and filter those 50 candidates down to the top 5. This solves the trade-off between speed and accuracy: you get the speed of vector search across millions of documents combined with the deep semantic understanding of a heavy transformer model.

I learned this when building an enterprise document search. Vector search returned files containing similar words, but missed the specific answer. Adding a reranking step raised the accuracy of the top 3 results from 60% to 92% with only a 50ms latency increase.

### The Real-World Analogy
The easiest way I understand this is to compare recruiting to a two-stage filter. You receive 1,000 resumes. Your HR department runs a quick keyword filter (stage 1) to find the top 30 candidates who mention the required coding language. Then, your engineering team reviews those 30 resumes in detail (stage 2 reranking) to select the top 3 candidates to interview.

### The Code
```python
# Interfacing with a reranking model using a mock cross-encoder client
class MockRerankerClient:
    def __init__(self, model_name: str = "cohere-rerank-v3"):
        self.model_name = model_name

    def rerank(self, query: str, documents: list[str], top_n: int = 3) -> list[str]:
        # In production, this makes an API call to Cohere/BGE reranker
        # Simulates scoring each document relative to the query context
        scored_docs = []
        for doc in documents:
            # Cross-encoder evaluates query and document jointly
            score = self._compute_cross_encoder_score(query, doc)
            scored_docs.append((score, doc))
        
        # Sort by score descending and return the top N documents
        scored_docs.sort(key=lambda x: x[0], reverse=True)
        return [doc for _, doc in scored_docs[:top_n]]

    def _compute_cross_encoder_score(self, query: str, doc: str) -> float:
        # Mock calculation: overlap of keywords
        query_words = set(query.lower().split())
        doc_words = set(doc.lower().split())
        return len(query_words.intersection(doc_words)) / len(query_words)
```

### Interview Questions
- Why are Cross-Encoders more accurate than Bi-Encoders for calculating document relevance?
- What are the operational latency costs of adding a reranker to your RAG pipeline?
- Explain the difference between bi-encoder embeddings and cross-encoder scoring.

### References
- [Pinecone: Rerankers Guide](https://www.pinecone.io/learn/cohere-rerank/)
- [Hugging Face: Cross-Encoder Models](https://huggingface.co/cross-encoder)

## Agentic RAG and Self-Correction Loops

### Definition & The Problem It Solves
Agentic RAG gives the LLM active control over the retrieval process. Instead of running a fixed search-and-generate script, the model is equipped with database search tools. It decides *if* it needs to search the database, evaluates the quality of the retrieved chunks, and determines if it needs to rephrase its query and run a second search. This solves the problem of models generating incorrect answers when the initial database query retrieves incomplete or noisy data.

When I built a medical reference tool, the initial vector search often missed specific dosage rules. Upgrading to Agentic RAG allowed the model to analyze the initial results, realize it was missing key info, and trigger a second search for the specific dosage tables.

### The Real-World Analogy
I like to compare this to hiring a detective. Standard RAG is like telling a researcher: "Retrieve files 1, 2, and 3, and write a summary". If the files do not contain the answer, they summarize what little they have. Agentic RAG is like telling the detective: "Find out where the suspect was on Tuesday". The detective goes to the archive, reads a file, realizes it is missing, searches another drawer, and does not report back until they have verified the details.

### The Code
```python
# A self-correcting agent loop structure that evaluates retrieval quality
import json

class AgenticRetriever:
    def __init__(self, database_tool, llm):
        self.db = database_tool
        self.llm = llm

    def run_query(self, user_question: str, max_attempts: int = 3) -> str:
        current_query = user_question
        
        for attempt in range(max_attempts):
            # 1. Search database
            chunks = self.db.search(current_query)
            
            # 2. Ask LLM to evaluate if the retrieved chunks contain the answer
            evaluation_prompt = (
                f"Question: {user_question}\n"
                f"Retrieved Context: {json.dumps(chunks)}\n"
                "Evaluate if this context is sufficient to answer the question. "
                "Respond in JSON format: {'sufficient': bool, 'new_search_query': str or null}"
            )
            
            evaluation = json.loads(self.llm.call(evaluation_prompt))
            
            if evaluation["sufficient"]:
                # If sufficient, generate final response
                generation_prompt = f"Answer this question: {user_question} using context: {json.dumps(chunks)}"
                return self.llm.call(generation_prompt)
            
            # If not, update query and retry the search loop
            current_query = evaluation["new_search_query"]
            
        return "Search failed: Context could not be resolved."
```

### Interview Questions
- What are the safety risks of allowing an agentic loop to run database queries recursively?
- How do you design tool descriptions to ensure an agent correctly triggers a search tool?
- How do you implement a token budget to prevent an agent loop from burning money on retries?

### References
- [Self-RAG: Learning to Retrieve, Generate, and Critique (Asai et al., 2023)](https://arxiv.org/abs/2310.11511)
- [LangChain: Concept of Self-RAG](https://langchain-ai.github.io/langgraph/tutorials/rag/langgraph_self_rag/)
