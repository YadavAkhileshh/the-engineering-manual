# 03 Vector Databases and Graph RAG

This guide covers vector storage and GraphRAG: embedding model selection, chunking and overlap strategies, pgvector vs dedicated vector databases, HNSW indexes, and GraphRAG architectures.

## Embedding Models Selection

### Definition & The Problem It Solves
Embedding Models convert text into vectors. Selecting an embedding model requires balancing cost, dimensional size, context length, and deployment constraints. Commercial models (like OpenAI's `text-embedding-3-small` or `text-embedding-3-large`) are serverless, but incur cost-per-token and network latency. Open-source models (like `BAAI/bge-m3` or `sentence-transformers/all-MiniLM-L6-v2`) run locally, allowing for free, offline, low-latency execution, but require hosting infrastructure and GPU memory resources.

When I deployed my first global search tool, I chose a high-dimension model without thinking. My vector storage costs tripled, and my retrieval latency spiked. Standardizing to a model with dimensions scaled down using Matryoshka representation learning resolved the balance.

### The Real-World Analogy
I like to compare choosing an embedding model to renting delivery vans. OpenAI is like calling a courier service (pay-per-package, easy, zero maintenance, but expensive if you send 10,000 packages daily). An open-source model is like buying a delivery truck for your company (free to run after purchase, completely private, but you must build the garage and pay for the driver).

### The Code
```python
# Comparison loading a local HuggingFace embedding and calling OpenAI
import os
from sentence_transformers import SentenceTransformer
from openai import OpenAI

class EmbeddingsSelector:
    def __init__(self):
        # Local open-source model
        self.local_model = SentenceTransformer("all-MiniLM-L6-v2")
        # OpenAI client
        self.client = OpenAI(api_key=os.getenv("OPENAI_API_KEY", "mock-key"))

    def get_local_embedding(self, text: str) -> list[float]:
        # Runs locally on CPU/GPU
        embedding = self.local_model.encode(text)
        return embedding.tolist()

    def get_openai_embedding(self, text: str) -> list[float]:
        # Serverless API call
        response = self.client.embeddings.create(
            model="text-embedding-3-small",
            input=text
        )
        return response.data[0].embedding
```

### Interview Questions
- What is Matryoshka Representation Learning and how does it optimize vector database index sizes?
- Why would you choose a local embedding model over a cloud API service for enterprise data?
- Explain how the sequence length limit of an embedding model impacts your document retrieval.

### References
- [MTEB: Massive Text Embedding Benchmark](https://huggingface.co/spaces/mteb/leaderboard)
- [OpenAI: New embedding models announcement](https://openai.com/index/new-embedding-models-and-api-updates/)

## Document Chunking and Overlap Strategies

### Definition & The Problem It Solves
Chunking is the process of splitting large documents into smaller text fragments before generating embeddings. If you embed a whole 100-page document as a single vector, the specific details get lost in the averaged coordinates. Standard chunking strategies include **Fixed-Size** (character or token counts), **Recursive** (splitting at paragraphs, then sentences), and **Semantic** (splitting where meaning changes). **Overlap** (duplicating words between consecutive chunks) is necessary to preserve context boundaries so details at the split points are not lost.

I learned this when users searched for concepts split across page transitions. Without overlap, the model separated the noun from its description, making the chunk unretrievable.

### The Real-World Analogy
The easiest way I understand this is to visualize cutting a film reel into short clips. If you cut the film exactly every 5 minutes (fixed chunking), you will slice right through the middle of a sentence a character is speaking. Adding overlap is like making sure each clip contains the last 10 seconds of the previous clip so the viewer does not lose the thread of the conversation.

### The Code
```python
# Implementing recursive character chunking with overlap
class RecursiveTextChunker:
    def chunk_text(self, text: str, chunk_size: int = 100, overlap: int = 20) -> list[str]:
        # Visual Example of text slice boundary overlap:
        # Chk 1: [This is a very long doc... overlap start]
        # Chk 2: [overlap start... rest of chunk]
        
        chunks = []
        start = 0
        text_length = len(text)
        
        if text_length <= chunk_size:
            return [text]
            
        while start < text_length:
            end = start + chunk_size
            chunk = text[start:end]
            chunks.append(chunk)
            
            # Move start pointer back by overlap size for next iteration
            start += (chunk_size - overlap)
            if start >= text_length - overlap:
                break # Avoid creating tiny trailing chunks
                
        return chunks
```

### Interview Questions
- Explain how too small or too large chunk sizes degrade RAG performance.
- Why is semantic chunking computationally more expensive than recursive character chunking?
- How does overlap help preserve relationships between words?

### References
- [Pinecone: Chunking Strategies for LLM Applications](https://www.pinecone.io/learn/chunking-strategies/)

## Postgres and pgvector vs Dedicated Vector Databases

### Definition & The Problem It Solves
A Vector Database stores and indexes high-dimensional vectors for fast similarity searches. For full-stack developers, **pgvector** (a Postgres extension) is often the best starting point because it allows storing vectors directly in columns next to relational user data, avoiding the complexity of syncing data across two databases. Dedicated vector stores (like Pinecone, Weaviate, or Qdrant) solve scale issues: they are built from the ground up for billions of vectors, providing high-performance query speeds and advanced metadata filtering.

When I built a prototype, I set up a dedicated vector database cluster. I spent more time writing cron jobs to sync user record changes to the vector store than writing AI features. Migrating to pgvector eliminated this synchronization logic.

### The Real-World Analogy
I like to compare pgvector to adding a bicycle rack to your car. You already own the car (Postgres), and adding the rack (pgvector) lets you carry bicycles (vectors) without buying a new vehicle. A dedicated vector database is like buying a specialized delivery truck: it is unnecessary if you only carry two bikes, but essential if you run a bicycle distribution warehouse.

### The Code
```python
# Storing and querying vector data using SQL and pgvector syntax concepts
# Note: Requires pgvector extension enabled in PostgreSQL
import psycopg2

class PostgresVectorDB:
    def __init__(self, connection_string: str):
        self.conn = psycopg2.connect(connection_string)

    def search_similar_documents(self, query_embedding: list[float], limit: int = 5) -> list[dict]:
        with self.conn.cursor() as cur:
            # The <=> operator in pgvector computes cosine distance
            sql = "SELECT id, content, embedding <=> %s::vector AS distance FROM documents ORDER BY distance ASC LIMIT %s;"
            cur.execute(sql, (query_embedding, limit))
            results = cur.fetchall()
            
            return [{"id": r[0], "content": r[1], "distance": r[2]} for r in results]
```

### Interview Questions
- Why does storing vectors in pgvector simplify transaction safety (ACID) compared to external databases?
- Under what scaling conditions should you migrate from pgvector to a dedicated vector store like Pinecone?
- What are the differences between Exact Nearest Neighbor search and Approximate Nearest Neighbor (ANN) search?

### References
- [pgvector GitHub Repository](https://github.com/pgvector/pgvector)
- [Supabase: Vector Search in Postgres](https://supabase.com/docs/guides/database/vector-columns)

## Hierarchical Navigable Small World (HNSW) Indexing

### Definition & The Problem It Solves
HNSW (Hierarchical Navigable Small World) is an Approximate Nearest Neighbor (ANN) algorithm. Without an index, finding similar vectors requires comparing your query vector against every single vector in the database (a linear scan), which takes seconds or minutes at scale. HNSW solves this by creating a multi-layered graph network where vectors are linked based on proximity, allowing search queries to skip linear scans and navigate to the nearest neighbor in logarithmic time.

I learned this when our vector table reached 100,000 items, causing query latency to spike to 400ms. Building an HNSW index brought search latency back down to 10ms.

### The Real-World Analogy
The easiest way I understand this is to compare HNSW to a highway network on a map. The top layer represents the interstate highway: you make large jumps between major cities. The middle layer represents state roads: you get closer to your destination town. The bottom layer contains neighborhood streets: you navigate directly to the specific house. You find the location without driving down every road in the state.

### The Code
```python
# Configuring an HNSW index on a Postgres table using pgvector
# This illustrates the index creation parameters (m, ef_construction)
class HNSWIndexConfigurator:
    @staticmethod
    def get_index_creation_sql(table_name: str, column_name: str) -> str:
        # m: Max connections per node in the graph
        # ef_construction: Size of the dynamic candidate list for index building
        return (
            f"CREATE INDEX ON {table_name} "
            f"USING hnsw ({column_name} vector_cosine_ops) "
            f"WITH (m = 16, ef_construction = 64);"
        )
```

### Interview Questions
- How does the `ef_search` parameter affect search recall accuracy and query latency?
- Explain the trade-offs of using HNSW indexes in terms of RAM consumption and write (insert) speeds.
- What is the difference between IVFFlat and HNSW index algorithms?

### References
- [Efficient and Robust ANN Search Using HNSW Graphs (Malkov & Yashunin, 2016)](https://arxiv.org/abs/1603.09320)

## GraphRAG and Knowledge Graph Extraction

### Definition & The Problem It Solves
GraphRAG combines vector databases with Knowledge Graphs. Standard vector RAG fails at "global" queries (e.g. "What are the main themes of all customer feedback?") because it only retrieves isolated chunks containing similar words. GraphRAG solves this by using an LLM to extract entities (people, places, concepts) and their relationships from your data, building a connected graph, and executing community detection algorithms to summarize global themes before answering.

When I built a compliance review tool, standard RAG missed patterns because the matching facts were spread across different reports. Implementing GraphRAG allowed the model to connect events via shared entities.

### The Real-World Analogy
I like to compare standard RAG to looking at individual census files to understand a town: you pull five folders and read about specific families, but miss the big picture. GraphRAG is like drawing a family tree and social network map connecting all residents. By looking at the map's clusters, you immediately understand who controls the businesses and how the neighborhoods interact.

### The Code
```python
# Conceptual GraphRAG entity extraction step
import json

class GraphEntityExtractor:
    def __init__(self, llm_client):
        self.llm = llm_client

    def extract_relationships(self, chunk_text: str) -> list[dict]:
        prompt = (
            "Extract entities and their relationships from this text.\n"
            "Format the output as a JSON list of objects containing:\n"
            '{"source": "entity_name", "relation": "verb", "target": "entity_name"}'
        )
        
        raw_response = self.llm.call(prompt=prompt, context=chunk_text)
        try:
            relationships = json.loads(raw_response)
            return relationships
        except json.JSONDecodeError:
            return [] # Handle extraction errors gracefully
```

### Interview Questions
- Why does standard vector search fail to resolve global summarization queries?
- Explain Microsoft's GraphRAG pipeline from entity extraction to community summaries.
- When is standard Vector RAG preferred over GraphRAG in terms of indexing cost and search speed?

### References
- [Microsoft Research: GraphRAG blog post](https://www.microsoft.com/en-us/research/blog/graphrag-unlocking-llm-discovery-on-narrative-private-data/)
- [GraphRAG GitHub Repository](https://github.com/microsoft/graphrag)
