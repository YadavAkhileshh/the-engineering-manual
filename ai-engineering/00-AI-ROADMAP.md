# 00 AI Roadmap

This document outlines the engineering mindset transition and the learning roadmap to go from full-stack developer to AI application engineer.

## Software Engineering for Non-Deterministic Systems

### Definition & The Problem It Solves
In traditional web development, code is deterministic. If you pass parameters to a database save function, the record is saved, or an error is thrown. In AI engineering, large language models are non-deterministic, meaning the exact same input prompt can yield different text configurations on subsequent calls. AI engineering solves the challenge of wrapping these unpredictable model outputs in reliable, structured, and testable software systems.

When I first transitioned from writing backend Express routes to building LLM interfaces, I assumed I could treat the model like a standard REST API. I learned the hard way that standard unit testing fails when the server output changes daily, and that you must build validation layers to constrain these non-deterministic runtimes.

### The Real-World Analogy
I like to compare this to hiring an artisan baker for your restaurant. You write a strict recipe describing how to bake sourdough bread. Because the humidity changes and flour batches vary, the baker produces loaves that look slightly different every morning. Your backend software behaves like a quality control manager who measures each loaf with calipers and rejects the ones that are burnt before they reach the customers.

### The Code
```python
# Production-like wrapper validating non-deterministic API outputs
import time
from typing import Optional
from openai import OpenAI
from pydantic import BaseModel, ValidationError

class UserAction(BaseModel):
    action: str
    confidence: float

class ValidatedModelCaller:
    def __init__(self, api_key: str):
        self.client = OpenAI(api_key=api_key)

    def call_with_retry(self, prompt: str, max_retries: int = 3) -> Optional[UserAction]:
        for attempt in range(max_retries):
            try:
                # Direct call to LLM
                response = self.client.beta.chat.completions.parse(
                    model="gpt-4o-mini",
                    messages=[
                        {"role": "system", "content": "Extract the user action and confidence score."},
                        {"role": "user", "content": prompt}
                    ],
                    response_format=UserAction,
                )
                return response.choices[0].message.parsed
            except (ValidationError, Exception) as e:
                # Exponential backoff on failure
                time.sleep(2 ** attempt)
        return None
```

### Interview Questions
- How do you design integration tests for APIs that return natural language text?
- Explain how you handle model hallucination in a customer-facing workflow.
- Describe the strategies to mitigate API latency spikes from third-party model providers.

### References
- [OpenAI: Production Guide](https://platform.openai.com/docs/guides/production)
- [Microsoft: Testing Non-Deterministic Software](https://www.microsoft.com/en-us/research/publication/testing-non-deterministic-software/)

## The Path from Web Developer to AI Application Engineer

### Definition & The Problem It Solves
AI application engineering is distinct from machine learning research. A researcher spends years studying the mathematics of backpropagation, tensor operations, and neural architecture optimization to train base models. An AI application engineer uses software engineering patterns to ingest data, prompt models, route requests, orchestrate agent memory, and serve APIs. This division of labor prevents developers from feeling like they need a PhD in linear algebra just to build smart software products.

When I built my first RAG system, I wasted weeks reading academic papers about attention mechanisms before realizing that 90% of my production bugs were actually caused by bad database indexing and slow API network calls, not mathematical formulas.

### The Real-World Analogy
The easiest way I understand this is to compare it to database administration. You do not need to write database storage engines like InnoDB or Postgres from scratch in C++ to design a high-performance relational schema. You are a consumer of the database engine, and your job is to write clean queries, structure tables, and build indexes. Similarly, as an AI engineer, you consume existing weights and models, orchestrating them using Python and APIs.

### The Code
```python
# A typical AI engineering data pipeline step
# Reading, embedding, and saving text is an software engineering ingestion problem
import numpy as np

class EmbeddingsIngestor:
    def __init__(self, mock_db):
        self.mock_db = mock_db

    def mock_embed(self, text: str) -> np.ndarray:
        # Mocking an embedding model vector output (dimension 1536)
        # In production, this calls OpenAI or a local SentenceTransformer model
        np.random.seed(hash(text) % 2**32)
        vector = np.random.randn(1536)
        return vector / np.linalg.norm(vector)

    def process_and_store(self, document_id: str, content: str):
        if not content.strip():
            raise ValueError("Empty content")
        
        # Ingestion pipeline: clean, embed, and store
        cleaned_text = " ".join(content.split())
        vector = self.mock_embed(cleaned_text)
        
        self.mock_db.save({
            "id": document_id,
            "text": cleaned_text,
            "embedding": vector.tolist()
        })
```

### Interview Questions
- Why is a strong software engineering foundation more critical for AI development than deep ML knowledge?
- How do database indexing strategies translate to vector database retrieval speeds?
- What are the architectural differences between transactional databases (OLTP) and vector databases?

### References
- [Andrej Karpathy: Software 2.0](https://karpathy.medium.com/software-2-0-a64152b37c35)

## Learning Dependencies (Python to Agents)

### Definition & The Problem It Solves
Building AI systems requires a structured progression. Naively building multi-agent systems without understanding how chunking, embeddings, and context window limitations work leads to bloated codebases, high token bills, and fragile execution paths. Developing this knowledge sequentially ensures that you only introduce complex orchestration frameworks when you reach the limits of simpler prompting models.

When I first started, I jumped directly to building agent systems using LangChain, only to spend days debugging state loops. I realized I had no idea how the underlying tool-calling format was being passed to the API under the hood.

### The Real-World Analogy
I like to compare this to learning web development. You do not start by configuring Webpack, Server-Side Rendering, and Redux on day one. You start by writing clean HTML, CSS, and basic JavaScript. Once you hit the limitations of vanilla files, you introduce React. Once React requires state sharing, you introduce Redux. AI engineering behaves the same: master prompting and tokens before configuring vector databases, and master databases before introducing agent loops.

### The Code
```python
# Understanding the underlying HTTP request structure before using frameworks
import requests
import json

class RawModelRequester:
    def __init__(self, api_key: str):
        self.url = "https://api.openai.com/v1/chat/completions"
        self.headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {api_key}"
        }

    def request_chat(self, user_prompt: str) -> str:
        # Direct payload creation matches the API contract exactly
        payload = {
            "model": "gpt-4o-mini",
            "messages": [
                {"role": "user", "content": user_prompt}
            ],
            "temperature": 0.2
        }
        
        response = requests.post(self.url, headers=self.headers, data=json.dumps(payload))
        response.raise_for_status()
        return response.json()["choices"][0]["message"]["content"]
```

### Interview Questions
- What are the risks of using complex agent frameworks (e.g. LangChain) for simple data extraction tasks?
- Why is raw API integration often preferred over third-party framework wrappers in production?
- Explain how tool calling differs from naive text completions.

### References
- [DeepLearning.AI: Prompt Engineering for Developers](https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/)

## The 8-Week AI Engineering Syllabus

### Definition & The Problem It Solves
The 8-week syllabus organizes your learning path into structured milestones. It targets key blocks: foundations (weeks 1-2), context & vector stores (weeks 3-4), orchestration & frameworks (weeks 5-6), and deployment & security (weeks 7-8). This timeline structures your prep to avoid tutorial hell and focus on production-grade implementations.

When I created this syllabus, I wanted to map out a clear study plan that forces developers to construct real pipelines, measure their latency, and profile their token costs rather than just watching video tutorials.

### The Real-World Analogy
I like to compare this syllabus to a structured pilot training program. You do not climb into the cockpit of a commercial airliner (multi-agent orchestration) on your first hour. You start in a simulator learning the dashboard dials (tokens and prompting), move to single-engine planes (basic retrieval), and gradually work your way up to multi-engine jets with complex copilot instruments.

### The Code
```python
# A simple script representing week-by-week progress logging
# This acts as a CLI planner checking off modules
import os

class SyllabusTracker:
    def __init__(self):
        self.modules = {
            "Week 1-2": "Core: Transformers, tokens, and vector geometry",
            "Week 3-4": "Context: Prompt structures, Pydantic outputs, and chunking",
            "Week 5-6": "Pipelines: LlamaIndex ingest, LangGraph loops, and tools",
            "Week 7-8": "Production: serving (vLLM), security, and cost evaluations"
        }

    def print_plan(self):
        print("Syllabus Checklist:")
        for week, topic in self.modules.items():
            print(f"[{week}]: {topic}")

if __name__ == "__main__":
    tracker = SyllabusTracker()
    tracker.print_plan()
```

### Interview Questions
- What is the difference between semantic chunking and recursive text splitting?
- When does a RAG pipeline become more cost-effective than fine-tuning?
- Describe how to construct an automated evaluation pipeline for search relevance.

### References
- [LlamaIndex: Learning Roadmap](https://docs.llamaindex.ai/en/stable/getting_started/conceptual_overview/)
