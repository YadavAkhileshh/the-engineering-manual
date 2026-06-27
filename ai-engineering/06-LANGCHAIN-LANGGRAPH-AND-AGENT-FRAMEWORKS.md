# 06 LangChain, LangGraph, and Agent Frameworks

This guide covers LLM orchestration frameworks: the LangChain Expression Language (LCEL), tool calling mechanics, state machine architectures in LangGraph, Google ADK patterns, and CrewAI multi-agent collaboration.

## LangChain, Framework Hell, and LangChain Expression Language (LCEL)

### Definition & The Problem It Solves
LangChain is a general-purpose framework for orchestrating LLM applications. Senior developers often criticize it for "Framework Hell" (too many nested classes and fast-changing abstractions that make debugging difficult), but it remains popular for prototyping. **LangChain Expression Language (LCEL)** solves the complexity of nesting code by introducing a declarative way to chain components using the `|` (pipe) operator. This allows you to chain prompts, models, parser objects, and custom functions together, automatically handling streaming, async execution, and fallbacks.

When I first wrote custom pipelines, I spent hours writing try/except blocks to catch API failures and manage async calls. Switching to LCEL simplified this, handling the retry logic and routing automatically.

### The Real-World Analogy
I like to compare LCEL to an assembly line factory conveyor belt. Instead of a single worker picking up a raw part, painting it, packing it, and labeling it (standard Python code), you place the part on the belt. The belt carries it past station A (Prompt), station B (LLM), and station C (Parser). The `|` operator connects the stations so the output of one step drops directly into the input of the next.

### The Code
```python
# Declarative chain structure using LangChain Expression Language (LCEL)
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

class LCELChainBuilder:
    def __init__(self, llm_model):
        self.model = llm_model
        self.parser = StrOutputParser()

    def build_translation_chain(self) -> RunnablePassthrough:
        prompt = ChatPromptTemplate.from_template(
            "Translate this text to French: {input_text}"
        )
        
        # The | operator chains prompt -> model -> output parser
        chain = (
            {"input_text": RunnablePassthrough()} 
            | prompt 
            | self.model 
            | self.parser
        )
        return chain
```

### Interview Questions
- Why do developers complain about "Framework Hell" when building complex LangChain applications?
- Explain how the `|` operator in LCEL handles async execution and batch processing.
- How do you implement custom runnables using the `@tool` or `RunnableLambda` interfaces?

### References
- [LangChain: LCEL Conceptual Guide](https://python.langchain.com/docs/concepts/lcel/)

## Tool Calling Mechanics

### Definition & The Problem It Solves
Tool Calling (Function Calling) is the process where an LLM decides to execute external code. Models cannot perform actions directly: they cannot query databases, write files, or check APIs. Tool calling solves this: you define your Python function schemas (parameters, descriptions) and pass them to the API. The model returns a structured `tool_calls` request containing the function name and arguments it wants to run. Your Python backend parses this request, executes the actual code locally, and sends the output back to the model to generate the final response.

I learned how critical this was when users asked: "What is my order status?". Instead of trying to write a regex to parse the question, tool calling allows the model to generate a structured database query automatically.

### The Real-World Analogy
The easiest way I understand this is to compare the LLM to a senior architect visiting a construction site. The architect does not carry tools or pour concrete. They inspect the plans and tell the builder (your Python code): "Use the shovel tool to dig a hole here, 3 feet deep." The builder digs the hole, measures the depth, and reports back: "The hole is dug, it is exactly 3 feet deep." The architect then decides the next step based on that feedback.

### The Code
```python
# Handling a raw OpenAI tool calling request loop
import json
from openai import OpenAI

def get_current_weather(location: str) -> str:
    # The actual Python function executed locally on the server
    return f"The weather in {location} is 72 degrees and sunny."

class ToolCallingCoordinator:
    def __init__(self, api_key: str):
        self.client = OpenAI(api_key=api_key)
        self.tools = [{
            "type": "function",
            "function": {
                "name": "get_current_weather",
                "description": "Get the current weather for a location",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {"type": "string", "description": "The city and state"}
                    },
                    "required": ["location"]
                }
            }
        }]

    def execute_query(self, user_prompt: str) -> str:
        # Step 1: Send query and tool schemas to LLM
        response = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": user_prompt}],
            tools=self.tools
        )
        message = response.choices[0].message
        
        # Step 2: Check if model wants to call a tool
        if message.tool_calls:
            tool_call = message.tool_calls[0]
            args = json.loads(tool_call.function.arguments)
            
            # Step 3: Run the local function
            result = get_current_weather(args["location"])
            
            # Step 4: Send the tool result back to the model to get the final response
            final_response = self.client.chat.completions.create(
                model="gpt-4o-mini",
                messages=[
                    {"role": "user", "content": user_prompt},
                    message,
                    {"role": "tool", "content": result, "tool_call_id": tool_call.id}
                ]
            )
            return final_response.choices[0].message.content
            
        return message.content
```

### Interview Questions
- Why must you provide detailed descriptions for tool schemas and parameters?
- How does tool calling differ from standard text completion?
- What security vulnerabilities are introduced when allowing an LLM to call database write functions?

### References
- [OpenAI: Function Calling](https://platform.openai.com/docs/guides/function-calling)
- [Anthropic: Tool Use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)

## State Management and Cycles in LangGraph

### Definition & The Problem It Solves
LangGraph is a state-management library for building agents. Standard chains are Directed Acyclic Graphs (DAGs), meaning execution flows in a single direction and cannot loop back. Real agent workflows require loops (cycles)—for example, searching a database, realizing the search failed, and looping back to try a different query. LangGraph solves this by structuring agents as state machines: you define **Nodes** (Python functions that execute code) and **Edges** (conditional logic routing flow based on the current state), managing a shared state object across all steps.

When I built a web researcher agent using linear code, the agent regularly got stuck when it hit a broken URL. Moving to LangGraph allowed me to write a loop: if a URL failed to load, the agent returned to the search node to find a different link.

### The Real-World Analogy
I like to compare LangGraph to a board game. The board has spaces (Nodes) and directions on how to move (Edges). The players have cards in their hands (State). You roll the dice and land on a space. You read the space's card: if your score is below 10 (conditional edge), you must go back three spaces (a cycle) and try again. The game continues in loops until you reach the winning space.

### The Code
```python
# A simple LangGraph agent structure managing state and conditional routing
from typing import Annotated, TypedDict
from langgraph.graph import StateGraph, START, END

# Define the shared state structure
class AgentState(TypedDict):
    query: str
    search_results: list[str]
    is_sufficient: bool

def search_database_node(state: AgentState) -> dict:
    # Node executing database search
    query = state["query"]
    # Simulating a search
    results = [f"Result matching '{query}'"]
    return {"search_results": results}

def check_sufficiency_node(state: AgentState) -> dict:
    # Node evaluating search results quality
    results = state["search_results"]
    # Simple check: fail if results contain specific keywords
    is_ok = len(results) > 0 and "fail" not in results[0]
    return {"is_sufficient": is_ok}

def route_flow(state: AgentState) -> str:
    # Conditional edge routing the next step based on state
    if state["is_sufficient"]:
        return END # Stop execution
    return "rephrase_node" # Loop back to rephrase query

def rephrase_node(state: AgentState) -> dict:
    # Node modifying query for next iteration
    new_query = f"alternative {state['query']}"
    return {"query": new_query}

# Build the state graph
builder = StateGraph(AgentState)
builder.add_node("search_db", search_database_node)
builder.add_node("check_quality", check_sufficiency_node)
builder.add_node("rephrase_node", rephrase_node)

builder.add_edge(START, "search_db")
builder.add_edge("search_db", "check_quality")
builder.add_conditional_edges("check_quality", route_flow)
builder.add_edge("rephrase_node", "search_db") # Loop cycle

graph = builder.compile()
```

### Interview Questions
- Why are cycles necessary for building autonomous agents compared to standard chains?
- How does LangGraph manage concurrent execution conflicts when multiple nodes write to the same state field?
- What is the difference between an Edge and a Conditional Edge in a state graph?

### References
- [LangGraph: Concepts documentation](https://langchain-ai.github.io/langgraph/concepts/high_level/)

## Google Agent Development Kit (ADK) Architectures

### Definition & The Problem It Solves
The Google Agent Development Kit (ADK) is an event-driven framework for building agents using Google's models and services. While LangGraph uses state graphs and nodes, the Google ADK focuses on event subscriptions and tools. It solves the challenge of connecting agents directly to enterprise resources (like Google Sheets, Drive, or Gmail) while maintaining strict access controls and session isolation, providing a clean path for building corporate assistants.

When I built a file management agent with LangGraph, managing the OAuth tokens and permissions for Google Workspace took more code than the agent itself. Using the Google ADK simplified this, handling the workspace permissions natively.

### The Real-World Analogy
The easiest way I understand this is to compare the Google ADK to a hotel concierge. Instead of the concierge running errands themselves (LangGraph state engine), they use the hotel's existing internal systems: they call the valet for your car, order food from the kitchen, and bill it directly to your room card. They orchestrate services without leaving their desk.

### The Code
```python
# Conceptual structure of a Google ADK event-driven agent registration
class GoogleADKAgent:
    def __init__(self, model_name: str = "gemini-1.5-pro"):
        self.model = model_name
        self.tools = []

    def register_tool(self, tool_func):
        # Register local tools
        self.tools.append(tool_func)

    def handle_event(self, event: dict) -> str:
        # Event-driven handler routing inputs to tool actions
        user_message = event.get("text", "")
        # Process message and determine tool execution
        return f"Agent processed event with model {self.model} using tools {self.tools}"
```

### Interview Questions
- How does the event-driven model of the Google ADK differ from the state machine model of LangGraph?
- Explain the security controls available when deploying ADK agents in corporate environments.
- How do you manage long-term agent memory across Google Workspace sessions?

### References
- [Google Cloud: Building Agents with Gemini](https://cloud.google.com/gemini/docs/api/agents)

## CrewAI and Collaborative Multi-Agent Systems

### Definition & The Problem It Solves
CrewAI is a framework for orchestrating teams of collaborative agents. In complex workflows (like research and writing), a single agent struggles because it must switch between different roles, which leads to mistakes and low-quality output. CrewAI solves this by allowing you to define multiple agents with distinct roles, goals, and backstories, assign specific tasks to each agent, and define how they pass work and collaborate with each other.

I learned this when building a blog writer bot. A single agent was terrible at verifying links and often wrote articles using broken references. Splitting the work between a "Researcher Agent" (who only verified URLs) and a "Writer Agent" (who only wrote copy) resolved the issue.

### The Real-World Analogy
I like to compare CrewAI to a movie production crew. You do not ask the director to write the script, set up the lights, act, and edit the film. You hire a screenwriter to write the script (Agent 1), hand it to the director to run the set (Agent 2), and pass the footage to the editor to compile the final cut (Agent 3). Each person has a specific job, passing their output to the next crew member.

### The Code
```python
# Multi-agent crew setup: Researcher and Writer collaboration schema
class MockCrewAgent:
    def __init__(self, role: str, goal: str, backstory: str):
        self.role = role
        self.goal = goal
        self.backstory = backstory

class MockCrewTask:
    def __init__(self, description: str, agent: MockCrewAgent):
        self.description = description
        self.agent = agent

class MultiAgentCrewCoordinator:
    def __init__(self, agents: list[MockCrewAgent], tasks: list[MockCrewTask]):
        self.agents = agents
        self.tasks = tasks

    def kickoff(self) -> str:
        # Executes tasks sequentially, passing output from task A as input to task B
        current_context = ""
        for task in self.tasks:
            agent = task.agent
            # Execute step simulating agent logic using context
            current_context = self._execute_agent_step(agent, task.description, current_context)
        return current_context

    def _execute_agent_step(self, agent: MockCrewAgent, task_desc: str, context: str) -> str:
        return f"[{agent.role}]: Resolved task '{task_desc}' using input context: {context[:50]}"
```

### Interview Questions
- How do multi-agent systems reduce context window costs compared to single-agent systems?
- What are the common failure modes of multi-agent loops, and how do you prevent infinite conversations between agents?
- Explain the difference between hierarchical and sequential multi-agent orchestration.

### References
- [CrewAI: Getting Started](https://docs.crewai.com/)
