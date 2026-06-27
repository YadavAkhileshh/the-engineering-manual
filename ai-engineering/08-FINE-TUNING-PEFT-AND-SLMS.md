# 08 Fine-Tuning, PEFT, and SLMs

This guide covers model customization: full vs parameter-efficient fine-tuning (LoRA/QLoRA), the Hugging Face library stack, training dataset formats, Small Language Models (SLMs), and local model execution.

## Why Fine-Tune: Custom Knowledge vs Style Formatting

### Definition & The Problem It Solves
Fine-Tuning is the process of adjusting a pre-trained model's internal weights by training it on a custom dataset. Developers often try to use fine-tuning to teach a model new facts, but this is a mistake: models are bad at recalling facts from fine-tuning alone (which is why we use RAG). Fine-tuning solves formatting and efficiency problems: it teaches a model a specific output style, forces adherence to complex JSON schemas, reduces token costs by eliminating long system prompts, and adapts the model to specialized terminology or internal code structures.

When I deployed an extraction pipeline, our system prompts were 3,000 tokens long because they contained multiple examples. Fine-tuning a smaller model on these examples allowed me to remove the system prompt entirely, cutting our token bills by 80% while keeping the same output quality.

### The Real-World Analogy
I like to compare fine-tuning to hiring an experienced accountant. Pre-training is like their college education: they already know math and basic tax rules. If you need them to find facts in your receipts, you do not retrain their brain (fine-tuning); you hand them the receipts folder (RAG). However, if you want them to write reports using your company's custom spreadsheet layout, you train them on that style (fine-tuning) so they write reports correctly without constant guidance.

### The Code
```python
# Formatting training data for fine-tuning as JSONL messages
import json

class FineTuningDataFormatter:
    def format_conversation_jsonl(self, raw_examples: list[dict], output_file: str):
        # Open a JSONL file to write structured training pairs
        with open(output_file, "w") as f:
            for ex in raw_examples:
                # Format matches OpenAI/HuggingFace conversational formats
                training_pair = {
                    "messages": [
                        {"role": "system", "content": "You are a database formatter."},
                        {"role": "user", "content": ex["input"]},
                        {"role": "assistant", "content": ex["target_json"]}
                    ]
                }
                f.write(json.dumps(training_pair) + "\n")
```

### Interview Questions
- Why is RAG preferred over fine-tuning for building models that query dynamic business facts?
- How does fine-tuning reduce API latency and per-request token costs in production?
- What are the risks of model overfitting when fine-tuning on a small dataset?

### References
- [OpenAI: Fine-Tuning Guide](https://platform.openai.com/docs/guides/fine-tuning)
- [Anyscale: Fine-Tuning vs RAG](https://www.anyscale.com/blog/fine-tuning-llms-or-rag-how-to-choose-the-right-approach)

## Parameter-Efficient Fine-Tuning (PEFT) and LoRA/QLoRA

### Definition & The Problem It Solves
Full Fine-Tuning requires updating all billions of parameters in a model, which requires massive clusters of expensive GPUs. Parameter-Efficient Fine-Tuning (PEFT) solves this infrastructure bottleneck. The most popular method, **LoRA (Low-Rank Adaptation)**, freezes the base model weights and inserts small, trainable parameter matrices into the attention layers. **QLoRA (Quantized LoRA)** takes this further by quantizing the base model to 4-bit precision, allowing you to fine-tune a model on a single consumer GPU instead of a data center cluster.

When I first planned to train a 13-billion parameter model, our cloud quote was thousands of dollars for multi-GPU instances. Switching to QLoRA allowed me to train the exact same model on a single RTX GPU for under twenty dollars.

### The Real-World Analogy
The easiest way I understand this is to compare the model to a massive university textbook. Full fine-tuning is like erasing and rewriting paragraphs on every single page of the textbook: it takes forever and is extremely expensive. LoRA is like keeping the textbook completely closed and attaching a small sticky note to the cover containing your custom page adjustments. You read the textbook through the corrections on the sticky note.

### The Code
```python
# Configuring a LoRA training setup using HuggingFace PEFT
# Note: Illustrates configuration parameters (r, lora_alpha, target_modules)
from peft import LoraConfig, get_peft_model
from transformers import AutoModelForCausalLM

class LoRAConfigurator:
    def __init__(self, model_id: str):
        # Load base model (in production, loaded in 8-bit or 4-bit for QLoRA)
        self.base_model = AutoModelForCausalLM.from_pretrained(model_id)

    def apply_lora_adapter(self) -> get_peft_model:
        # Define LoRA parameters
        # r: Rank dimension (lower rank = fewer trainable parameters)
        # lora_alpha: Scaling factor for the adapter weights
        lora_config = LoraConfig(
            r=8,
            lora_alpha=32,
            target_modules=["q_proj", "v_proj"], # target attention layers
            lora_dropout=0.05,
            bias="none",
            task_type="CAUSAL_LM"
        )
        
        # Wrap base model with LoRA adapter layers
        peft_model = get_peft_model(self.base_model, lora_config)
        return peft_model
```

### Interview Questions
- What is the difference between LoRA and QLoRA in terms of GPU memory utilization?
- How does the rank parameter (r) impact the representation capacity of the LoRA adapter?
- Why do you target projection matrices (e.g. q_proj, v_proj) for adapter insertions?

### References
- [LoRA: Low-Rank Adaptation of Large Language Models (Hu et al., 2021)](https://arxiv.org/abs/2106.09685)
- [QLoRA: Efficient Finetuning of Quantized LLMs (Dettmers et al., 2023)](https://arxiv.org/abs/2305.14314)

## The Hugging Face Stack (Transformers, Peft, TRL, Datasets)

### Definition & The Problem It Solves
The Hugging Face stack is the standard open-source library collection for working with models. It solves the fragmentation of model tooling by providing four unified libraries: **`transformers`** (loads model weights and tokenizers), **`peft`** (configures parameter adapters), **`datasets`** (manages training datasets), and **`trl`** (provides the `SFTTrainer` for supervised training). Together, they standardize the training pipeline: you load a dataset, wrap a model in LoRA, and run a single command to execute training.

When I started working with open-source models, every provider had a different script for loading weights. Using the Hugging Face stack allowed me to load models from different providers (like Meta, Mistral, or Google) using the exact same code templates.

### The Real-World Analogy
I like to compare the Hugging Face stack to building a kit car. `transformers` is the engine and chassis (the base model); `peft` is the custom body kit (the adapter); `datasets` is the fuel tank and fuel supply; and `trl` is the assembly line and mechanics who bolt all the pieces together and tune the engine.

### The Code
```python
# Supervised Fine-Tuning (SFT) script layout using HuggingFace TRL
# Note: Conceptual representation of the SFTTrainer training loop
from datasets import load_dataset
from transformers import AutoTokenizer, TrainingArguments
from trl import SFTTrainer

class ModelTrainerWrapper:
    def __init__(self, model_id: str, dataset_path: str):
        self.tokenizer = AutoTokenizer.from_pretrained(model_id)
        self.dataset = load_dataset("json", data_files=dataset_path)
        self.model_id = model_id

    def configure_and_train(self, output_dir: str):
        # Configure output paths and learning parameters
        training_args = TrainingArguments(
            output_dir=output_dir,
            per_device_train_batch_size=4,
            gradient_accumulation_steps=4,
            learning_rate=2e-4,
            logging_steps=10,
            max_steps=100
        )
        
        # In production, self.model_id is passed wrapped with LoraConfig
        # SFTTrainer handles loading data and running backward propagation loops
        trainer = SFTTrainer(
            model=self.model_id,
            train_dataset=self.dataset["train"],
            dataset_text_field="text",
            max_seq_length=512,
            tokenizer=self.tokenizer,
            args=training_args
        )
        
        # trainer.train() # Executes model training loop
```

### Interview Questions
- Explain the role of gradient accumulation steps in Hugging Face training arguments.
- What does the Hugging Face `AutoModel` class do and how does it load model weights dynamically?
- How do you save and merge a trained LoRA adapter back into a base model?

### References
- [Hugging Face: Transformers Documentation](https://huggingface.co/docs/transformers/index)
- [Hugging Face: TRL Supervised Fine-Tuning Trainer](https://huggingface.co/docs/trl/sft_trainer)

## Small Language Models (SLMs) and Edge Computing

### Definition & The Problem It Solves
Small Language Models (SLMs) are models with fewer parameters (typically 1B to 3B parameters, such as Llama 3.2 1B/3B, Microsoft Phi-3, or Google Gemma). Traditional LLMs (with 70B+ parameters) are hosted on expensive cloud GPU clusters. SLMs solve this: they are small enough to run locally on mobile phones, laptops, and edge devices, providing fast, private, offline, and cheap text processing.

When I designed a mobile writing assistant, cloud API latency fluctuated between one and three seconds depending on the network. Running a 1B parameter model locally on the user's phone cut latency to under 100ms and eliminated server costs.

### The Real-World Analogy
The easiest way I understand this is to compare models to vehicles. A large 70B model is like a train: it can carry anything and is extremely powerful, but it requires a massive station and tracks (cloud servers). An SLM is like a scooter: it cannot carry a shipping container, but it is cheap to run, fits in your garage, and lets you zip around the neighborhood without waiting for a schedule.

### The Code
```python
# Loading a small language model locally using HuggingFace
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

class LocalSLMRunner:
    def __init__(self, model_id: str = "meta-llama/Llama-3.2-1B-Instruct"):
        self.tokenizer = AutoTokenizer.from_pretrained(model_id)
        # Load model using half-precision (FP16) on GPU if available
        device = "cuda" if torch.cuda.is_available() else "cpu"
        self.model = AutoModelForCausalLM.from_pretrained(
            model_id, 
            torch_dtype=torch.float16 if device == "cuda" else torch.float32
        ).to(device)
        self.device = device

    def generate_local_response(self, prompt: str) -> str:
        inputs = self.tokenizer(prompt, return_tensors="pt").to(self.device)
        with torch.no_grad():
            outputs = self.model.generate(**inputs, max_new_tokens=50)
            
        return self.tokenizer.decode(outputs[0], skip_special_tokens=True)
```

### Interview Questions
- Why are Small Language Models (SLMs) more practical for offline edge deployments than large models?
- What are the limitations of a 1B parameter model compared to a 70B parameter model?
- How does the context window size of an SLM compare to modern cloud-hosted models?

### References
- [Meta Llama 3.2 Announcement](https://llama.meta.com/llama3_2/)
- [Microsoft Phi-3 Technical Report](https://arxiv.org/abs/2404.14219)

## Local Execution using Ollama

### Definition & The Problem It Solves
Ollama is a lightweight tool that packages, runs, and serves open-source models locally. Managing raw model weights, configuring quantizations, and setting up serving APIs is complex. Ollama solves this by packaging open-source models into single files (using GGUF format) and providing a simple REST API and command-line interface, allowing developers to download and run models locally with a single terminal command.

When I wanted to run local tests during development, I had to write complex scripts to load and run model checkpoints. Installing Ollama allowed me to run a model in the background and connect to it using a standard OpenAI-compatible API.

### The Real-World Analogy
I like to compare Ollama to Docker for models. Instead of manually downloading code repositories, installing python packages, and setting up server ports, Ollama acts as a container runner: you type `ollama run llama3.2` and it downloads the packaged model, configures it for your laptop's hardware, and sets up a local API endpoint automatically.

### The Code
```python
# Interfacing with a local Ollama server using python requests
import requests
import json

class OllamaLocalClient:
    def __init__(self, base_url: str = "http://localhost:11434"):
        self.url = f"{base_url}/api/chat"

    def ask_local_model(self, prompt: str, model_name: str = "llama3.2") -> str:
        payload = {
            "model": model_name,
            "messages": [
                {"role": "user", "content": prompt}
            ],
            "stream": False # Disable streaming for simple response
        }
        
        response = requests.post(self.url, data=json.dumps(payload))
        response.raise_for_status()
        return response.json()["message"]["content"]
```

### Interview Questions
- How does Ollama optimize memory usage by unloading models when they are inactive?
- What is the difference between Ollama's local API endpoints and standard OpenAI endpoints?
- How do you configure a custom Modelfile in Ollama to set custom system prompts?

### References
- [Ollama Official Website](https://ollama.com/)
- [Ollama GitHub Repository](https://github.com/ollama/ollama)
