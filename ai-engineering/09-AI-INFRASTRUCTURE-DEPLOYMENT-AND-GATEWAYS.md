# 09 AI Infrastructure, Deployment, and Gateways

This guide covers model hosting and routing: inference vs training bottlenecks, model quantization (GGUF), serving engines (vLLM, SGLang), API proxy gateways (LiteLLM), and FastAPI token streaming.

## Inference vs Training and GPU Execution

### Definition & The Problem It Solves
Training is the process of learning model weights; Inference is the process of running a trained model to generate outputs. In production, AI engineers focus on Inference. While training is a batch process run occasionally, inference runs millions of times daily. Inference is the system bottleneck: it requires high GPU memory bandwidth to load weights into memory for every token generated, explaining why managing memory and optimizing throughput is critical for keeping server costs low.

When I deployed my first application, I assumed our training server would make a good hosting server. I quickly learned that hosting a model requires different GPU configurations: instead of raw compute power, you need high memory bandwidth to serve multiple users concurrently.

### The Real-World Analogy
I like to compare training to writing a massive, authoritative encyclopedia: it takes years, a team of researchers, and enormous resources. Inference is like hiring a librarian to answer questions using that encyclopedia: the book is already written, and the librarian's speed depends on how fast they can flip the pages and read the text aloud to visitors.

### The Code
```python
# Simulating the difference: training gradient loops vs fast inference runs
import numpy as np

class GPUExecutionMock:
    def run_training_step(self, inputs: np.ndarray, targets: np.ndarray) -> float:
        # Training calculates gradients, weights updates, and backpropagation
        # High compute cost (FLOPs)
        predictions = inputs * 0.5
        loss = np.mean((predictions - targets) ** 2)
        gradient = 2 * (predictions - targets) / len(inputs)
        # Weights are mutated here (backward pass)
        return float(loss)

    def run_inference_step(self, inputs: np.ndarray) -> np.ndarray:
        # Inference only runs the forward pass
        # Memory-bandwidth bottleneck: loading weights to calculate predictions
        frozen_weights = np.array([0.5])
        predictions = inputs * frozen_weights
        return predictions
```

### Interview Questions
- Why is memory bandwidth the primary bottleneck during LLM inference?
- How does the ratio of compute to memory operations (arithmetic intensity) change between training and inference?
- What is the purpose of the KV Cache in optimizing inference execution?

### References
- [LLM Inference Series: The Bottleneck](https://kipp.ly/transformer-inference-arithmetic-intensity/)

## Quantization Formats (FP16, INT8, INT4, GGUF)

### Definition & The Problem It Solves
Quantization is the process of compressing model weights by reducing the numerical precision of their parameters (e.g. converting 16-bit floating-point numbers - FP16, into 8-bit or 4-bit integers - INT8/INT4). This solves the GPU memory bottleneck: a 70-billion parameter model requires 140GB of VRAM in FP16, but compressing it to 4-bit allows it to run on a single 40GB GPU. **GGUF** is a file format designed to run these quantized models efficiently on CPU and GPU architectures.

When I first hosted a model, we had to lease two expensive GPUs to fit the uncompressed weights. Compressing the model to 4-bit allowed us to host it on a single, cheap server with almost no loss in output quality.

### The Real-World Analogy
The easiest way I understand this is to compare quantization to compressing a high-resolution photo. The raw photo is massive and takes forever to load on a website. If you compress it to a JPEG (quantization), you lose some pixel details if you zoom in, but the photo loads instantly, looks great, and fits on a standard phone screen.

### The Code
```python
# Conceptual simulation of converting 32-bit floats to 8-bit integers
import numpy as np

class Quantizer:
    def quantize_to_int8(self, weights: np.ndarray) -> tuple[np.ndarray, float]:
        # Scale float values to fit within the signed 8-bit integer range [-128, 127]
        max_val = np.max(np.abs(weights))
        scale = 127.0 / max_val
        
        # Multiply by scale and round to nearest integer
        quantized_weights = np.round(weights * scale).astype(np.int8)
        return quantized_weights, scale

    def dequantize(self, quantized_weights: np.ndarray, scale: float) -> np.ndarray:
        # Restore approximation of original float values
        return quantized_weights.astype(np.float32) / scale
```

### Interview Questions
- What is the difference between Post-Training Quantization (PTQ) and Quantization-Aware Training (QAT)?
- How does quantization impact the perplexity and accuracy of a language model?
- Why is the GGUF format preferred over GGML for local CPU model execution?

### References
- [Hugging Face: Introduction to Quantization](https://huggingface.co/docs/optimum/concept_guides/quantization)
- [llama.cpp GitHub Repository](https://github.com/ggerganov/llama.cpp)

## High-Throughput Serving Engines (vLLM, SGLang, TGI)

### Definition & The Problem It Solves
Serving Engines are frameworks designed to host models in production. Running models using basic libraries (like Hugging Face `pipeline`) is too slow for production because they process requests one by one. Serving engines like **vLLM** introduce **PagedAttention** (which optimizes memory by managing the KV cache like virtual RAM pages, reducing memory waste from 60% to 4%), while **SGLang** introduces RadixAttention and speculative decoding. These engines allow servers to process dozens of requests in parallel, multiplying throughput.

I learned this when our demo crashed during a user test. The basic API server queued requests sequentially, causing response times to spike to 30 seconds. Replacing it with vLLM allowed the server to handle 20 users simultaneously without lag.

### The Real-World Analogy
I like to compare a basic model runner to a restaurant with a single cook who prepares one meal at a time: if ten tables order, table ten waits hours. vLLM is like a master chef running a high-speed kitchen: they cook multiple steaks on one grill, prepare salads while the meat rests, and share ingredients across orders to serve everyone at the same time.

### The Code
```python
# Conceptual serving setup using vLLM command execution
# Serving engines are typically deployed as standalone Docker containers
class vLLMCommandBuilder:
    @staticmethod
    def get_run_command(model_id: str, port: int = 8000) -> str:
        # host the model using OpenAI compatible API routing
        # --gpu-memory-utilization limits VRAM consumption
        return (
            f"python -m vllm.entrypoints.openai.api_server "
            f"--model {model_id} "
            f"--port {port} "
            f"--gpu-memory-utilization 0.90 "
            f"--max-model-len 4096"
        )
```

### Interview Questions
- How does PagedAttention solve the KV cache memory fragmentation problem?
- What are the differences between static batching and continuous (dynamic) batching?
- Explain how speculative decoding improves inference speed.

### References
- [vLLM: PagedAttention paper (Kwon et al., 2023)](https://arxiv.org/abs/2309.06180)
- [SGLang GitHub Repository](https://github.com/sgl-project/sglang)

## Unified AI Gateways (LiteLLM and Portkey Routing)

### Definition & The Problem It Solves
An AI Gateway is a proxy server that sits between your application and model APIs. Hardcoding specific SDKs (like `openai.OpenAI()`) creates a single point of failure: if OpenAI goes down, your app crashes. AI gateways like **LiteLLM** or **Portkey** solve this by providing a unified API format: you send requests to the gateway, and it handles load balancing, automatic failover (e.g. routing to Anthropic if OpenAI fails), rate limiting, cost tracking, and centralized logging.

When OpenAI experienced an outage last year, our entire application went offline. Setting up LiteLLM allowed the gateway to automatically failover to Anthropic, keeping our app online with zero manual intervention.

### The Real-World Analogy
The easiest way I understand this is to compare an AI gateway to a universal remote control. Instead of having five different remotes for your TV, soundbar, and console, you use one remote. If you replace your TV (changing your model provider), you do not buy a new remote; you program the universal remote to route the signals to the new screen without changing how you press the buttons.

### The Code
```python
# Routing requests through a mock LiteLLM proxy gateway
# The gateway translates a single request format to multiple providers
import requests

class UnifiedGatewayClient:
    def __init__(self, gateway_url: str = "http://localhost:4000"):
        self.url = f"{gateway_url}/chat/completions"

    def send_completion(self, user_prompt: str, model_route: str) -> str:
        # The API payload matches the standard OpenAI format
        # model_route dictates which backend API the gateway targets (e.g., anthropic/claude-3)
        payload = {
            "model": model_route,
            "messages": [
                {"role": "user", "content": user_prompt}
            ]
        }
        
        # Gateway handles authentications and routing internally
        response = requests.post(self.url, json=payload)
        response.raise_for_status()
        return response.json()["choices"][0]["message"]["content"]
```

### Interview Questions
- Why is an AI gateway preferred over hardcoding model SDKs in enterprise applications?
- How do you implement cost-based routing rules inside an API gateway?
- Describe how an AI gateway handles rate limiting and token bucket algorithms.

### References
- [LiteLLM: Proxy Server Documentation](https://docs.litellm.ai/docs/proxy_server)
- [Portkey: AI Gateway Feature List](https://portkey.ai/features/ai-gateway)

## The AI Backend with FastAPI

### Definition & The Problem It Solves
FastAPI is an asynchronous Python web framework designed to handle high-concurrency connections. Traditional synchronous web frameworks (like Flask or Django) block the execution thread for each incoming request. In AI engineering, where model API calls or local generation steps take several seconds, hosting an LLM backend on Flask causes the server's thread pool to exhaust quickly, causing request timeouts for subsequent users. FastAPI solves this by running an event loop: it releases execution threads during network wait states (like waiting for OpenAI API bytes), allowing a single server instance to process thousands of concurrent users waiting for model outputs. It also integrates with Pydantic for schema validations and supports StreamingResponse to push tokens to the frontend in real time using Server-Sent Events (SSE).

When I first deployed a chatbot backend on Flask, three concurrent users generating responses froze the API for everyone else. Rewriting the routing handlers to use FastAPI's async endpoints immediately restored throughput.

### The Real-World Analogy
I like to compare Flask to a restaurant with a single waiter who takes your order, walks into the kitchen, stands next to the stove waiting for the cook to prepare your steak (a blocking API call), and only returns to serve other tables once your meal is ready. FastAPI is like a waiter who takes your order, passes it to the kitchen, and immediately goes to assist other tables while your food is cooking, returning to serve you once the cook rings the bell.

### The Code
```python
# FastAPI production backend demonstrating Pydantic requests, SSE streaming,
# background tasks for long-running agents, and Dockerfile containerization.
import asyncio
import json
from uuid import uuid4
from fastapi import FastAPI, BackgroundTasks, HTTPException
from fastapi.responses import StreamingResponse
from pydantic import BaseModel, Field

app = FastAPI()

# InMemory database simulating task status tracking
task_store = {}

# 1. Pydantic request model for input validation and structured output mapping
class ChatRequest(BaseModel):
    prompt: str = Field(..., min_length=1, max_length=1000)
    temperature: float = Field(default=0.7, ge=0.0, le=1.0)

class TaskStatusResponse(BaseModel):
    task_id: str
    status: str
    result: str | None = None

# Helper simulating an agent execution loop
async def run_agent_workflow(task_id: str, prompt: str):
    task_store[task_id] = {"status": "processing", "result": None}
    
    # Simulate multi-step research agent steps
    for step in range(3):
        await asyncio.sleep(2) # Simulate tool execution latency
        
    task_store[task_id] = {
        "status": "completed",
        "result": f"Final agentic research report for: '{prompt}'"
    }

# 2. Async endpoint streaming tokens using Server-Sent Events (SSE)
async def event_generator(prompt: str):
    # Simulates streaming tokens from a model API response
    tokens = f"Transmitted tokens response for prompt: {prompt}".split()
    for token in tokens:
        # SSE messages must begin with 'data: ' and end with double newlines
        yield f"data: {json.dumps({'token': token})}\n\n"
        await asyncio.sleep(0.1)
    yield "data: [DONE]\n\n"

@app.post("/chat/stream")
async def chat_stream(request: ChatRequest):
    return StreamingResponse(
        event_generator(request.prompt),
        media_type="text/event-stream"
    )

# 3. Background task endpoint for long-running workflows (Polling Pattern)
@app.post("/agent/run", response_model=TaskStatusResponse)
async def start_agent_task(request: ChatRequest, background_tasks: BackgroundTasks):
    task_id = str(uuid4())
    task_store[task_id] = {"status": "queued", "result": None}
    
    # Hand off execution to background workers without blocking HTTP response
    background_tasks.add_task(run_agent_workflow, task_id, request.prompt)
    return TaskStatusResponse(task_id=task_id, status="queued")

@app.get("/agent/status/{task_id}", response_model=TaskStatusResponse)
async def get_agent_status(task_id: str):
    if task_id not in task_store:
        raise HTTPException(status_code=404, detail="Task not found")
    task_data = task_store[task_id]
    return TaskStatusResponse(
        task_id=task_id,
        status=task_data["status"],
        result=task_data["result"]
    )

# 4. Containerization blueprint: Dockerfile
# To run this container in production alongside model weights:
#
# FROM python:3.11-slim
# WORKDIR /app
# COPY requirements.txt .
# RUN pip install --no-cache-dir -r requirements.txt
# COPY . .
# # Expose port and run server
# EXPOSE 8000
# CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Interview Questions
- Why do blocking libraries (like standard requests) negate the performance benefits of FastAPI's async endpoints?
- Explain how `BackgroundTasks` in FastAPI differs from dedicated task queues like Celery or RabbitMQ.
- How do you parse and validate nested Pydantic models when mapping JSON data payloads?

### References
- [FastAPI: Concurrency and async/await](https://fastapi.tiangolo.com/async/)
- [Uvicorn: ASGI Web Server](https://www.uvicorn.org/)

