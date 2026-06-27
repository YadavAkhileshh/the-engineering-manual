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

## FastAPI Backend with Server-Sent Events (SSE) Streaming

### Definition & The Problem It Solves
Server-Sent Events (SSE) is a web standard that allows servers to stream real-time updates to browsers over a single HTTP connection. In AI apps, waiting for the model to generate a full 500-token response before sending it to the client takes 5-10 seconds, which is a terrible user experience. SSE solves this: as the serving engine generates each token, the FastAPI backend streams it to the client instantly, allowing the browser to render words as they are generated.

I learned this when testing our writing app. Users assumed the app was broken and refreshed the page because of the initial 5-second silence. Implementing SSE streaming made the app feel fast and responsive.

### The Real-World Analogy
I like to compare SSE streaming to watching a live ticker tape. Instead of a printing house printing a whole newspaper and mailing it to your house at the end of the day, a ticker tape printer prints each letter on a paper strip the second it is typed at the news station, allowing you to read the news word by word.

### The Code
```python
# FastAPI endpoint streaming tokens to the client using Server-Sent Events (SSE)
import asyncio
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

async def token_generator(prompt: str):
    # Simulates streaming tokens from a model generator
    words = f"This is the streamed response to: {prompt}".split()
    for word in words:
        # SSE format requires prepending data: to each message chunk
        yield f"data: {json.dumps({'token': word})}\n\n"
        await asyncio.sleep(0.15) # Simulate token generation delay
        
    yield "data: [DONE]\n\n"

@app.get("/stream")
async def stream_tokens(prompt: str):
    # StreamingResponse sets transfer-encoding chunked headers automatically
    return StreamingResponse(token_generator(prompt), media_type="text/event-stream")
```

### Interview Questions
- How does the `text/event-stream` media type differ from standard `application/json` responses?
- What are the differences between WebSockets and Server-Sent Events for streaming model responses?
- How do you handle connection drops on the client side when receiving SSE streams?

### References
- [FastAPI: Streaming Responses](https://fastapi.tiangolo.com/advanced/custom-response/#streamingresponse)
- [MDN: Using server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)
