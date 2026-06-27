# 02 Context Engineering and Structured Outputs

This guide covers context management: transition to context engineering, strict system prompt contracts, few-shot layouts, chain-of-thought, structured outputs, JSON validation using Pydantic and the instructor library, and XML parsing patterns.

## Context Engineering and System Prompts

### Definition & The Problem It Solves
Prompt Engineering is often misunderstood as writing clever adjectives to cajole a model into working. Context Engineering treats the prompt as an orchestration contract, configuring system instructions, memory histories, and tools. System Prompts act as immutable execution guidelines that define the model's role, constraints, and safety guardrails, solving the problem of models drifting from their task or responding in conversational filler when they should be acting like software utilities.

When I first wrote chatbot scripts, users bypassed my rules by typing "ignore previous instructions". Implementing context boundaries where the system prompt is strictly separated from user-generated values solved this vulnerability.

### The Real-World Analogy
I like to compare a system prompt to a notary public's official employment contract. You do not just ask them to sign things; you hand them a legal document stating: "You are a notary public. You only sign documents if the user presents a driver's license. You must reject any request that is not witnessed. You will print 'REJECTED' if validation fails." This contract defines their boundaries before they ever speak to a customer.

### The Code
```python
# System prompt contract setup with strict role isolation and validation
import os
from openai import OpenAI

class SystemContractClient:
    def __init__(self):
        self.client = OpenAI(api_key=os.getenv("OPENAI_API_KEY", "mock-key"))

    def call_with_system_contract(self, user_query: str) -> str:
        # Separate instructions strictly from raw user queries
        system_prompt = (
            "ROLE: You are an API gateway routing utility.\n"
            "INSTRUCTIONS:\n"
            "- Read the user query and output only one word: 'DATABASE' or 'SUPPORT'.\n"
            "- If the query is ambiguous, default to 'SUPPORT'.\n"
            "- Do not write explanations, markdown headers, or sentences. Output one word only."
        )
        
        response = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": f"USER_INPUT: {user_query}"}
            ],
            temperature=0.0 # Force determinism
        )
        return response.choices[0].message.content.strip()
```

### Interview Questions
- What is the difference between system-level messages and user-level messages in LLM APIs?
- How do system prompts protect against direct prompt injection attacks?
- Why is it recommended to set model Temperature to 0.0 when testing system prompt instructions?

### References
- [Anthropic: System Prompt Guidelines](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/system-prompts)

## Few-Shot and Chain of Thought Prompting

### Definition & The Problem It Solves
Few-Shot Prompting injects example inputs and outputs directly into the context window, showing the model how to behave rather than just explaining it. Chain of Thought (CoT) forces the model to generate intermediate reasoning steps (often wrapped in `<think>` tags) before outputting the final answer. These patterns solve accuracy issues in reasoning-heavy tasks, like math or SQL generation, where models fail if they are forced to output a final answer immediately.

When I built a SQL generation endpoint, the model regularly generated invalid schemas. By introducing three few-shot examples showing schema definitions followed by valid SQL queries, syntax errors dropped to zero.

### The Real-World Analogy
The easiest way I understand this is to compare few-shot to showing a developer a company's codebase before they write their first pull request. Instead of explaining the coding standards in a long document, you show them three complete pull requests. Chain of thought is like asking a mathematician to solve a complex equation on a chalkboard: you instruct them to show every intermediate calculation step rather than just writing the final number.

### The Code
```python
# Few-shot prompts with step-by-step reasoning validation
class ReasoningGenerator:
    def build_few_shot_prompt(self, problem: str) -> list[dict]:
        messages = [
            {
                "role": "system",
                "content": "You are a mathematical assistant. Explain your reasoning steps inside <think> tags, then print the final answer."
            },
            # Few-shot example 1
            {"role": "user", "content": "Calculate the sum of factors of 6."},
            {
                "role": "assistant",
                "content": "<think>\n1. Factors of 6 are 1, 2, 3, and 6.\n2. Sum them: 1 + 2 + 3 + 6 = 12.\n</think>\n12"
            },
            # Actual task
            {"role": "user", "content": problem}
        ]
        return messages
```

### Interview Questions
- Why does Chain of Thought prompting improve performance on reasoning tasks?
- What are the trade-offs of using Chain of Thought in terms of execution latency and token costs?
- How does Few-Shot prompting help models output custom formatting patterns?

### References
- [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models (Wei et al., 2022)](https://arxiv.org/abs/2201.11903)

## Structured Outputs and Constraint Sampling

### Definition & The Problem It Solves
Structured Outputs guarantee that a model returns data conforming to a specific schema (like JSON). Before 2024, developers had to parse raw string responses using regex, handling frequent errors when models returned trailing commas or conversational wrappers (like "Here is your JSON:"). Modern APIs solve this through **Constraint Sampling**, where the inference engine adjusts its token probabilities at the hardware level, setting the probability of any token that violates the JSON schema to zero.

I remember writing loops that retried API calls five times because the model returned invalid JSON syntax. Switching to constraint-sampled structured outputs resolved this issue.

### The Real-World Analogy
I like to compare this to filling out a tax form. Without constraints (standard generation), you hand the user a blank sheet of paper and ask for their income. They might write "I make fifty grand a year" or draw a picture. With constraint sampling, you hand them a form where the boxes physically lock out letters: they can only type numerical digits in the income field.

### The Code
```python
# OpenAI structured output enforcement using pydantic schemas
import os
from openai import OpenAI
from pydantic import BaseModel, Field

class DatabaseQueryPlan(BaseModel):
    table_name: str = Field(description="Target table to query")
    columns: list[str] = Field(description="List of columns to select")
    where_clause: str = Field(description="Filter condition")

class StructuredPlanner:
    def __init__(self):
        self.client = OpenAI(api_key=os.getenv("OPENAI_API_KEY", "mock-key"))

    def get_query_plan(self, request_text: str) -> DatabaseQueryPlan:
        # Constraint sampling is enforced by passing response_format
        response = self.client.beta.chat.completions.parse(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": "Create a structured database query plan based on the user request."},
                {"role": "user", "content": request_text}
            ],
            response_format=DatabaseQueryPlan
        )
        return response.choices[0].message.parsed
```

### Interview Questions
- How does constraint sampling differ from post-generation parsing?
- What is the difference between JSON Mode and Structured Outputs in modern APIs?
- Why do structured outputs sometimes increase the time-to-first-token latency?

### References
- [OpenAI: Structured Outputs](https://platform.openai.com/docs/guides/structured-outputs)

## Pydantic Validation with the Instructor Library

### Definition & The Problem It Solves
The `instructor` library wraps LLM clients to automate Pydantic schema validation. When a model returns data, `instructor` parses it into a Pydantic model. If validation fails (e.g. a field does not pass Pydantic validator checks), `instructor` automatically catches the validation error, appends the error message to the context, and sends a retry request back to the model to correct its mistakes, solving the challenge of programmatic error handling inside API flows.

When I validated emails returned by models, they regularly hallucinated fake addresses. Wrapping my client with `instructor` allowed me to write custom Pydantic validators that automatically triggered self-correction loops.

### The Real-World Analogy
The easiest way I understand this is to compare it to a bank teller checking checks. You submit a check. If you forgot to write the date (Pydantic validation failure), the teller does not reject your account. Instead, they hand the check back, point to the missing date line (the error message), and ask you to fix it before they process it.

### The Code
```python
# Utilizing instructor with pydantic validators for auto-retrying failures
import instructor
from openai import OpenAI
from pydantic import BaseModel, Field, field_validator

class UserProfile(BaseModel):
    username: str = Field(description="Unique username")
    email: str = Field(description="Valid email address")

    @field_validator("email")
    @classmethod
    def validate_email_format(cls, value: str) -> str:
        if "@" not in value:
            raise ValueError("Email must contain '@' symbol")
        return value

class ValidatedProfileLoader:
    def __init__(self):
        # Patch the OpenAI client with instructor
        self.client = instructor.from_openai(OpenAI(api_key="mock-key"))

    def fetch_profile(self, user_description: str) -> UserProfile:
        # Max retries parameter configures the automatic self-correction loops
        profile: UserProfile = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "user", "content": f"Extract profile details: {user_description}"}
            ],
            response_model=UserProfile,
            max_retries=3
        )
        return profile
```

### Interview Questions
- How does the `instructor` library handle self-correction retry payloads?
- Why is Pydantic validation crucial for structured outputs?
- What are the risks of setting `max_retries` too high in an automated validation loop?

### References
- [Instructor Python Library Documentation](https://github.com/jxnl/instructor)

## XML Tagging for Complex Context Extraction

### Definition & The Problem It Solves
XML (Extensible Markup Language) tagging involves wrapping distinct context blocks in XML tags (e.g. `<document>`, `<instructions>`). Models, particularly Anthropic's Claude, are trained to parse XML tags. This solves the problem of model confusion when processing mixed context: by separating system instructions, source documents, and user inputs, the model understands exactly where one document ends and another begins.

When I passed a mixed transcript containing chat history and instructions, the model sometimes executed instructions that were actually typed by the customer in the chat logs. Separating user inputs inside `<chat_history>` tags prevented this exploit.

### The Real-World Analogy
Imagine a courier delivery box containing three items: a contract, a feedback form, and a set of instructions. Instead of tossing them loose into the box where they get shuffled, you place each item in a distinct folder labeled "Contract", "Feedback", and "Instructions". The recipient can now locate and read each item without confusing their contents.

### The Code
```python
# Formatting prompts using XML tags to structure complex inputs
class XMLPromptFormatter:
    def format_extraction_prompt(self, document_content: str, rules: list[str]) -> str:
        # Group variables and rules within explicit tags
        rules_xml = "\n".join([f"<rule>{r}</rule>" for r in rules])
        
        prompt = (
            "Extract entity information from the source document following these rules.\n"
            f"<rules>\n{rules_xml}\n</rules>\n"
            f"<document>\n{document_content}\n</document>\n"
            "Output your findings inside <extracted_entities> tags."
        )
        return prompt
```

### Interview Questions
- Why do transformer models parse XML tags better than JSON tags in long-context tasks?
- How do you parse output XML tags programmatically from a model's raw text response?
- Explain how XML tags prevent command leakage when processing untrusted document data.

### References
- [Anthropic: Use XML Tags](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags)
