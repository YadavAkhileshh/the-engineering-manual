# 05 LlamaIndex Deep Dive

This guide covers the LlamaIndex framework: comparison with LangChain, document ingestion pipelines, indexing options, advanced query engines, routing setups, event-driven Workflows, and production persistence.

## LlamaIndex vs LangChain Architecture

### Definition & The Problem It Solves
LlamaIndex is an orchestration framework specifically designed for data ingestion, indexing, and Retrieval-Augmented Generation (RAG). Developers often mistake it for a LangChain clone, but they serve different purposes. LangChain is a general-purpose toolkit for building sequential chains of action. LlamaIndex solves the data-to-LLM bridge: it focuses on data loaders, semantic indices, and high-performance search retrieval, making it the preferred choice when your primary engineering challenge is connecting complex databases or document stores to an LLM.

When I built a search tool for a customer database, I originally used LangChain. I spent days writing custom loaders and indexing wrappers. Rebuilding the tool with LlamaIndex cut our codebase in half because LlamaIndex handles document ingestion natively.

### The Real-World Analogy
I like to compare LangChain to a general contractor's toolbox containing hammers, saws, and measuring tape to build any house structure. LlamaIndex is like a specialized plumbing and water filtration installation kit: you could try to build a water filter using tools from the general toolbox, but using the dedicated plumbing kit ensures the water flows smoothly and cleanly with much less effort.

### The Code
```python
# A clean LlamaIndex setup to load documents and run a simple query
# Demonstrating native LlamaIndex semantic search abstractions
import os
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader

class SimpleLlamaIndexSearch:
    def __init__(self, data_directory: str):
        # Configure API key environment variable
        os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY", "mock-key")
        self.data_dir = data_directory

    def query_documents(self, user_question: str) -> str:
        # SimpleDirectoryReader automatically detects file formats (txt, pdf, docx)
        documents = SimpleDirectoryReader(self.data_dir).load_data()
        
        # Ingestion, embedding, and vector store indexing occur in one line
        index = VectorStoreIndex.from_documents(documents)
        
        # Build query engine and execute
        query_engine = index.as_query_engine()
        response = query_engine.query(user_question)
        return str(response)
```

### Interview Questions
- Under what conditions would you choose LlamaIndex over LangChain for a production application?
- Explain the role of the Document and Node abstractions in LlamaIndex.
- How does LlamaIndex integrate with third-party vector databases like Pinecone?

### References
- [LlamaIndex: Conceptual Overview](https://docs.llamaindex.ai/en/stable/getting_started/conceptual_overview/)

## Data Ingestion and LlamaParse Pipelines

### Definition & The Problem It Solves
Data Ingestion is the process of loading, parsing, and cleaning raw files before indexing. Standard PDF parsers extract raw text but lose formatting structure, turning tables, columns, and charts into unreadable paragraphs. LlamaIndex solves this through its collection of **Data Connectors** and **LlamaParse** (a cloud-based parsing engine). LlamaParse parses complex PDFs (such as financial statements or manuals), extracts tables as markdown, and describes visual charts, converting them into structured text that the LLM can query.

I learned this when our search tool returned incorrect numbers from financial reports. Standard PDF parsers mixed up the columns in our balance sheets. Routing those documents through LlamaParse preserved the table structures, ensuring accurate retrieval.

### The Real-World Analogy
The easiest way I understand this is to compare parsing to scanning receipts. A standard parser is like running the receipts through a paper shredder and reading the words in a line: you get the prices, but lose track of which item belongs to which price. LlamaParse is like a data entry assistant who reads each receipt and copies it into a clean spreadsheet, preserving the rows, columns, and totals.

### The Code
```python
# Declarative ingestion pipeline loading, chunking, and embedding documents
from llama_index.core import Document
from llama_index.core.node_parser import SentenceSplitter
from llama_index.core.ingestion import IngestionPipeline
from llama_index.embeddings.openai import OpenAIEmbedding

class IngestionPipelineManager:
    def __init__(self):
        # Configure embedding step and splitting rules
        self.pipeline = IngestionPipeline(
            transformations=[
                SentenceSplitter(chunk_size=512, chunk_overlap=50),
                OpenAIEmbedding(model="text-embedding-3-small")
            ]
        )

    def run_pipeline(self, raw_documents: list[Document]) -> list:
        # Executes splitting and embedding steps sequentially
        nodes = self.pipeline.run(documents=raw_documents)
        return nodes # Returns chunked nodes with embedding vectors attached
```

### Interview Questions
- Why do standard PDF parsers fail to extract data from multi-column layouts?
- How does LlamaParse convert tables and visual charts into formats suitable for LLM parsing?
- Explain how you handle file parsing errors in a high-volume data pipeline.

### References
- [LlamaIndex: LlamaParse documentation](https://docs.llamaindex.ai/en/stable/module_guides/loading/connector/llama_parse/)
- [LlamaIndex: Ingestion Pipeline guide](https://docs.llamaindex.ai/en/stable/module_guides/loading/ingestion_pipeline/)

## Index Architectures (VectorStoreIndex vs KnowledgeGraphIndex)

### Definition & The Problem It Solves
An Index is the structured database format LlamaIndex uses to store and retrieve data. Choosing the right index architecture is critical. **VectorStoreIndex** embeds text chunks and retrieves them using vector similarity search, which is fast and handles specific keyword queries well. **KnowledgeGraphIndex** extracts entities and relationships from the text to build a connected network graph, solving the problem of answering complex, relational questions (e.g. "Who reports to whom in this organization?") that vector search struggles with.

When I built an onboarding wiki search, standard vector indexing failed to answer who was responsible for which product line because that data was spread across different articles. Building a Knowledge Graph Index resolved this.

### The Real-World Analogy
I like to compare a Vector Store Index to a library organized by book subjects: if you want books about cooking, you go to the cooking section. A Knowledge Graph Index is like a network map of the town showing all residents, their families, and where they work. If you want to know who is married to the mayor's sister, you trace the connection lines on the map instead of reading every biography in the library.

### The Code
```python
# Setting up and querying a VectorStoreIndex
from llama_index.core import VectorStoreIndex, Document

class IndexManager:
    def __init__(self, raw_text_data: list[str]):
        self.documents = [Document(text=t) for t in raw_text_data]

    def create_vector_index(self) -> VectorStoreIndex:
        # Build vector store index from raw documents
        index = VectorStoreIndex.from_documents(self.documents)
        return index

    def query_index(self, index: VectorStoreIndex, query: str) -> str:
        query_engine = index.as_query_engine()
        response = query_engine.query(query)
        return str(response)
```

### Interview Questions
- How does a SummaryIndex differ from a VectorStoreIndex in how it retrieves documents?
- What are the GPU and API cost trade-offs of building a KnowledgeGraphIndex compared to a VectorStoreIndex?
- How does LlamaIndex handle index updates when source documents change?

### References
- [LlamaIndex: Index Guides](https://docs.llamaindex.ai/en/stable/module_guides/indexing/)

## Advanced Query Engines (SubQuestionQueryEngine and Routers)

### Definition & The Problem It Solves
A Query Engine is the interface used to search and query your index. Advanced query engines solve complex search scenarios. A **RouterQueryEngine** routes queries to different datasources (e.g. sending math questions to a calculator tool, and history questions to a vector store). A **SubQuestionQueryEngine** breaks down a complex, multi-part query (e.g. "Compare company A's revenue in 2023 with company B's revenue") into sub-questions, queries the indexes for each sub-question, and compiles the results into a final answer.

I learned this when users typed comparison queries like "What is the difference between feature X and feature Y?". Standard search returned a mix of articles, but missed the specific comparisons. Implementing the sub-question engine split the query into two searches and combined the findings.

### The Real-World Analogy
The easiest way I understand this is to compare a query engine to a company receptionist. A standard engine is like a receptionist who handles every question by handing you the company brochure. A router engine is like a receptionist who directs you to the accounting department for billing questions, and to support for technical issues. A sub-question engine is like a project manager who receives a complex proposal request, assigns sections to different team members, gathers their reports, and writes a single response.

### The Code
```python
# Configuring a SubQuestionQueryEngine to resolve multi-part comparison queries
from llama_index.core import VectorStoreIndex, Document
from llama_index.core.tools import QueryEngineTool, ToolMetadata
from llama_index.core.query_engine import SubQuestionQueryEngine

class ComparisonEngineBuilder:
    def build_comparison_engine(self, doc_a: str, doc_b: str) -> SubQuestionQueryEngine:
        # 1. Create separate indices
        index_a = VectorStoreIndex.from_documents([Document(text=doc_a)])
        index_b = VectorStoreIndex.from_documents([Document(text=doc_b)])
        
        # 2. Wrap indices as tools
        tool_a = QueryEngineTool(
            query_engine=index_a.as_query_engine(),
            metadata=ToolMetadata(name="doc_a_tool", description="Information about Company A")
        )
        tool_b = QueryEngineTool(
            query_engine=index_b.as_query_engine(),
            metadata=ToolMetadata(name="doc_b_tool", description="Information about Company B")
        )
        
        # 3. Initialize SubQuestionQueryEngine
        sub_query_engine = SubQuestionQueryEngine.from_defaults(
            query_engine_tools=[tool_a, tool_b]
        )
        return sub_query_engine
```

### Interview Questions
- How does the SubQuestionQueryEngine decompose queries using the LLM before execution?
- Explain how RouterQueryEngine determines the best tool match using classification prompting.
- What are the latency impacts of running multiple sub-queries in parallel?

### References
- [LlamaIndex: Sub-Question Query Engine](https://docs.llamaindex.ai/en/stable/examples/query_engine/sub_question_query_engine/)
- [LlamaIndex: Router Query Engine](https://docs.llamaindex.ai/en/stable/module_guides/querying/router/)

## LlamaIndex Workflows and Production Persistence

### Definition & The Problem It Solves
LlamaIndex Workflows is an event-driven framework for building custom, step-by-step RAG logic. Standard query engines run as fixed scripts that are hard to modify. Workflows solve this by structuring logic as event loops: you define steps (Python functions) that trigger when they receive specific events, passing data through event objects. Additionally, **Production Persistence** allows you to save your embedded indices to disk and reload them on server startup, avoiding the cost and latency of re-embedding documents on every restart.

When I deployed my first production system, the server took 10 minutes to start because it re-embedded our document directories. Implementing storage persistence cut startup times to under a second.

### The Real-World Analogy
I like to compare Workflows to a manufacturing assembly line. Instead of a single worker doing every step (standard query engine), you set up stations. When a raw chassis arrives (StartEvent), station A paints it and passes it to station B (PaintEvent). Station B installs the engine and passes it to station C (EngineInstalledEvent). Each station only runs when the correct event lands on its workbench.

### The Code
```python
# Production pattern: persisting an index and reloading it from disk
import os
from llama_index.core import (
    VectorStoreIndex,
    SimpleDirectoryReader,
    StorageContext,
    load_index_from_storage
)

class ProductionIndexStore:
    def __init__(self, storage_dir: str):
        self.storage_dir = storage_dir

    def initialize_index(self, data_dir: str) -> VectorStoreIndex:
        # Check if index files exist on disk
        if os.path.exists(os.path.join(self.storage_dir, "docstore.json")):
            # Load index from local storage directory
            storage_context = StorageContext.from_defaults(persist_dir=self.storage_dir)
            index = load_index_from_storage(storage_context)
        else:
            # If not, build the index and save it to disk
            documents = SimpleDirectoryReader(data_dir).load_data()
            index = VectorStoreIndex.from_documents(documents)
            index.storage_context.persist(persist_dir=self.storage_dir)
            
        return index
```

### Interview Questions
- Why is an event-driven workflow model better than linear chain code for complex agent loops?
- How does `StorageContext` manage index persistence in LlamaIndex?
- What are the risks of loading a cached index if the underlying source files have been modified on disk?

### References
- [LlamaIndex: Workflows guide](https://docs.llamaindex.ai/en/stable/module_guides/workflowing/)
- [LlamaIndex: Storing indices](https://docs.llamaindex.ai/en/stable/module_guides/indexing/metadata_extraction/)
