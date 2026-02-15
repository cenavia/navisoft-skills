# Agent Patterns Reference

## Table of Contents
- [Simple Agent](#simple-agent)
- [ReAct Agent](#react-agent)
- [RAG Agent](#rag-agent)
- [Modular Agent](#modular-agent)
- [Orchestrator Agent](#orchestrator-agent)
- [Evaluator Agent](#evaluator-agent)

---

## Simple Agent

Basic agent with tools capability.

### File: src/agents/main.py

```python
"""
Simple agent with tool support.
"""

from typing import Literal

from langchain.chat_models import init_chat_model
from langchain_core.messages import AIMessage
from langchain_core.tools import tool
from langgraph.graph import StateGraph, START, END, MessagesState
from langgraph.prebuilt import ToolNode


# State definition
class State(MessagesState):
    """Agent state with message history."""
    pass


# Tools
@tool("get_current_time", description="Get the current date and time")
def get_current_time() -> str:
    """Returns the current date and time."""
    from datetime import datetime
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")


tools = [get_current_time]

# LLM configuration
llm = init_chat_model("openai:gpt-4o-mini", temperature=0)
llm_with_tools = llm.bind_tools(tools)


# Nodes
def assistant(state: State) -> dict:
    """Process user message and generate response."""
    response = llm_with_tools.invoke(state["messages"])
    return {"messages": [response]}


def should_continue(state: State) -> Literal["tools", "__end__"]:
    """Determine if tools should be called."""
    last_message = state["messages"][-1]
    if isinstance(last_message, AIMessage) and last_message.tool_calls:
        return "tools"
    return "__end__"


# Graph construction
builder = StateGraph(State)
builder.add_node("assistant", assistant)
builder.add_node("tools", ToolNode(tools))

builder.add_edge(START, "assistant")
builder.add_conditional_edges("assistant", should_continue)
builder.add_edge("tools", "assistant")

agent = builder.compile()
```

---

## ReAct Agent

Agent implementing the ReAct (Reasoning + Acting) pattern.

### File: src/agents/react.py

```python
"""
ReAct agent with reasoning and action loop.
"""

from langchain.agents import create_react_agent
from langchain.chat_models import init_chat_model
from langchain_core.tools import tool


@tool("search", description="Search for information on a topic")
def search(query: str) -> str:
    """Search for information about a query."""
    return f"Search results for: {query}"


@tool("calculate", description="Perform mathematical calculations")
def calculate(expression: str) -> str:
    """Evaluate a mathematical expression."""
    try:
        result = eval(expression)
        return str(result)
    except Exception as e:
        return f"Error: {e}"


tools = [search, calculate]

SYSTEM_PROMPT = """You are a helpful assistant that can search for information 
and perform calculations. Think step by step before taking action."""

agent = create_react_agent(
    model="openai:gpt-4o",
    tools=tools,
    prompt=SYSTEM_PROMPT,
)
```

---

## RAG Agent

Retrieval-Augmented Generation agent.

### File: src/agents/rag.py

```python
"""
RAG agent with document retrieval capabilities.
"""

from typing import List

from langchain.chat_models import init_chat_model
from langchain_core.documents import Document
from langchain_core.messages import SystemMessage
from langchain_core.prompts import PromptTemplate
from langgraph.graph import StateGraph, START, END, MessagesState


class State(MessagesState):
    """RAG agent state."""
    context: List[Document]
    query: str


SYSTEM_TEMPLATE = """\
You are a helpful assistant that answers questions based on the provided context.

Context:
{% for doc in context %}
{{ doc.page_content }}
---
{% endfor %}

If the context doesn't contain relevant information, say so.
"""

system_prompt = PromptTemplate.from_template(
    SYSTEM_TEMPLATE, 
    template_format="jinja2"
)

llm = init_chat_model("openai:gpt-4o-mini", temperature=0)


def retrieve(state: State) -> dict:
    """Retrieve relevant documents."""
    # TODO: Implement actual retrieval logic
    # This is a placeholder - integrate with your vector store
    query = state["messages"][-1].content
    docs = [Document(page_content="Placeholder context")]
    return {"context": docs, "query": query}


def generate(state: State) -> dict:
    """Generate response using retrieved context."""
    formatted_prompt = system_prompt.format(context=state["context"])
    messages = [SystemMessage(content=formatted_prompt)] + state["messages"]
    response = llm.invoke(messages)
    return {"messages": [response]}


# Graph construction
builder = StateGraph(State)
builder.add_node("retrieve", retrieve)
builder.add_node("generate", generate)

builder.add_edge(START, "retrieve")
builder.add_edge("retrieve", "generate")
builder.add_edge("generate", END)

agent = builder.compile()
```

---

## Modular Agent

Agent with modular structure for complex workflows. Customize nodes and routes for your specific domain.

### Directory Structure
```
src/agents/modular/
├── __init__.py
├── agent.py
├── state.py
├── nodes/
│   ├── __init__.py
│   ├── extractor/
│   │   ├── __init__.py
│   │   ├── node.py
│   │   └── prompt.py
│   ├── conversation/
│   │   ├── __init__.py
│   │   ├── node.py
│   │   └── prompt.py
│   └── action/
│       ├── __init__.py
│       ├── node.py
│       ├── prompt.py
│       └── tools.py
└── routes/
    ├── __init__.py
    └── intent/
        ├── __init__.py
        └── route.py
```

### File: src/agents/modular/__init__.py
```python
from agents.modular.agent import agent, make_graph

__all__ = ["agent", "make_graph"]
```

### File: src/agents/modular/state.py
```python
"""
Modular agent state definition.
Customize fields for your specific domain.
"""

from typing import Literal

from langgraph.graph import MessagesState


class State(MessagesState):
    """
    State for modular agent.
    
    Customize these fields based on your domain:
    - extracted_entities: Data extracted from user messages
    - current_intent: Detected user intent
    - context: Domain-specific context data
    """
    extracted_entities: dict
    current_intent: str
    context: dict
```

### File: src/agents/modular/agent.py
```python
"""
Modular agent graph definition.
"""

from typing import TypedDict

from langgraph.graph import StateGraph, START, END

from agents.modular.state import State
from agents.modular.nodes.extractor.node import extractor
from agents.modular.nodes.conversation.node import conversation
from agents.modular.nodes.action.node import action_node
from agents.modular.routes.intent.route import intent_route


def make_graph(config: TypedDict = None):
    """
    Create and compile the modular agent graph.
    
    Args:
        config: Optional configuration with checkpointer
        
    Returns:
        Compiled StateGraph
    """
    config = config or {}
    checkpointer = config.get("checkpointer", None)
    
    builder = StateGraph(State)
    
    # Add nodes
    builder.add_node("extractor", extractor)
    builder.add_node("conversation", conversation)
    builder.add_node("action", action_node)
    
    # Define edges
    builder.add_edge(START, "extractor")
    builder.add_edge("extractor", "conversation")
    builder.add_conditional_edges("conversation", intent_route)
    builder.add_edge("action", END)
    
    return builder.compile(checkpointer=checkpointer)


agent = make_graph()
```

### File: src/agents/modular/nodes/extractor/node.py
```python
"""
Extractor node for entity extraction.
Customize the schema for your domain entities.
"""

from typing import Optional

from langchain.chat_models import init_chat_model
from pydantic import BaseModel, Field

from agents.modular.state import State
from agents.modular.nodes.extractor.prompt import prompt_template


class ExtractedEntities(BaseModel):
    """
    Extracted entities from user message.
    Customize fields for your domain.
    """
    name: Optional[str] = Field(None, description="Name mentioned in message")
    email: Optional[str] = Field(None, description="Email address if provided")
    intent: Optional[str] = Field(None, description="Detected user intent")
    entities: dict = Field(default_factory=dict, description="Additional extracted entities")


llm = init_chat_model("openai:gpt-4o-mini", temperature=0)
structured_llm = llm.with_structured_output(schema=ExtractedEntities)


def extractor(state: State) -> dict:
    """
    Extract entities from user messages.
    
    Args:
        state: Current agent state
        
    Returns:
        Updated state with extracted entities
    """
    messages = state["messages"]
    last_message = messages[-1].content if messages else ""
    
    prompt = prompt_template.format(message=last_message)
    result = structured_llm.invoke(prompt)
    
    # Merge with existing entities
    existing = state.get("extracted_entities", {})
    extracted = {
        "name": result.name or existing.get("name"),
        "email": result.email or existing.get("email"),
        **result.entities,
    }
    
    return {
        "extracted_entities": extracted,
        "current_intent": result.intent or state.get("current_intent", ""),
    }
```

### File: src/agents/modular/nodes/extractor/prompt.py
```python
"""
Prompts for extractor node.
Customize entity types for your domain.
"""

from langchain_core.prompts import PromptTemplate

TEMPLATE = """\
Extract relevant entities from the following message.
If information is not present, leave the field as null.

Entity types to extract:
- name: Any person name mentioned
- email: Email address if provided
- intent: What the user wants to do
- entities: Any other relevant data as key-value pairs

Message: {{ message }}
"""

prompt_template = PromptTemplate.from_template(
    TEMPLATE,
    template_format="jinja2"
)
```

### File: src/agents/modular/nodes/conversation/node.py
```python
"""
Conversation node for general interaction.
Customize the system prompt for your domain.
"""

from langchain.chat_models import init_chat_model
from langchain_core.messages import SystemMessage

from agents.modular.state import State
from agents.modular.nodes.conversation.prompt import SYSTEM_PROMPT, format_context


llm = init_chat_model("openai:gpt-4o-mini", temperature=0.7)


def conversation(state: State) -> dict:
    """
    Handle general conversation with user.
    
    Args:
        state: Current agent state
        
    Returns:
        Updated state with assistant response
    """
    entities = state.get("extracted_entities", {})
    context = state.get("context", {})
    
    system_content = SYSTEM_PROMPT.format(
        user_context=format_context(entities, context)
    )
    
    messages = [SystemMessage(content=system_content)] + state["messages"]
    response = llm.invoke(messages)
    
    return {"messages": [response]}
```

### File: src/agents/modular/nodes/conversation/prompt.py
```python
"""
Prompts for conversation node.
Customize for your specific domain.
"""

SYSTEM_PROMPT = """\
You are a helpful AI assistant.

{user_context}

Be friendly, professional, and helpful.
Guide the user through their request.
"""


def format_context(entities: dict, context: dict) -> str:
    """
    Format extracted entities and context for the prompt.
    
    Args:
        entities: Extracted user entities
        context: Domain-specific context
        
    Returns:
        Formatted context string
    """
    lines = []
    
    if entities.get("name"):
        lines.append(f"User's name: {entities['name']}")
    
    for key, value in context.items():
        if value:
            lines.append(f"{key}: {value}")
    
    return "\n".join(lines) if lines else "No additional context available."
```

### File: src/agents/modular/nodes/action/node.py
```python
"""
Action node for domain-specific operations.
Customize tools and logic for your use case.
"""

from langchain.chat_models import init_chat_model
from langchain_core.messages import AIMessage
from langgraph.prebuilt import ToolNode

from agents.modular.state import State
from agents.modular.nodes.action.tools import tools
from agents.modular.nodes.action.prompt import SYSTEM_PROMPT


llm = init_chat_model("openai:gpt-4o-mini", temperature=0)
llm_with_tools = llm.bind_tools(tools)


def action_node(state: State) -> dict:
    """
    Execute domain-specific action.
    
    Args:
        state: Current agent state
        
    Returns:
        Updated state with action result
    """
    messages = state["messages"]
    context = state.get("context", {})
    entities = state.get("extracted_entities", {})
    
    # Build action context
    action_context = f"""
Current context: {context}
Extracted entities: {entities}
"""
    
    system_message = {"role": "system", "content": SYSTEM_PROMPT + action_context}
    response = llm_with_tools.invoke([system_message] + messages)
    
    return {"messages": [response]}
```

### File: src/agents/modular/nodes/action/prompt.py
```python
"""
Prompts for action node.
"""

SYSTEM_PROMPT = """\
You are an action executor. Use the available tools to complete the user's request.

Available actions depend on the tools configured for this agent.
Execute the appropriate action based on the user's intent.

"""
```

### File: src/agents/modular/nodes/action/tools.py
```python
"""
Tools for action node.
Customize with domain-specific tools.
"""

from langchain_core.tools import tool


@tool("execute_action", description="Execute a generic action with the given parameters")
def execute_action(action_type: str, parameters: dict) -> str:
    """
    Execute a domain-specific action.
    
    Args:
        action_type: Type of action to execute
        parameters: Action parameters
        
    Returns:
        Action result message
    """
    # TODO: Implement your domain-specific action logic
    return f"Action '{action_type}' executed with parameters: {parameters}"


@tool("get_information", description="Retrieve information based on a query")
def get_information(query: str) -> str:
    """
    Retrieve information from the system.
    
    Args:
        query: Information query
        
    Returns:
        Retrieved information
    """
    # TODO: Implement your information retrieval logic
    return f"Information for query '{query}': [Implement your logic here]"


# Export tools list - add your custom tools here
tools = [execute_action, get_information]
```

### File: src/agents/modular/routes/intent/route.py
```python
"""
Intent routing for modular agent.
Customize intents for your domain.
"""

from typing import Literal

from langchain.chat_models import init_chat_model
from pydantic import BaseModel, Field

from agents.modular.state import State


class RouteIntent(BaseModel):
    """
    Intent classification result.
    Customize the Literal types for your domain intents.
    """
    intent: Literal["conversation", "action"] = Field(
        "conversation",
        description="The detected user intent: 'conversation' for general chat, 'action' for specific operations"
    )


llm = init_chat_model("openai:gpt-4o-mini", temperature=0)
structured_llm = llm.with_structured_output(schema=RouteIntent)


def intent_route(state: State) -> Literal["action", "__end__"]:
    """
    Route based on user intent.
    
    Args:
        state: Current agent state
        
    Returns:
        Next node name or END
    """
    last_message = state["messages"][-1].content
    current_intent = state.get("current_intent", "")
    
    prompt = f"""Classify the user's intent from this message:
    
Message: {last_message}
Previously detected intent: {current_intent}

Determine if the user wants to:
- 'conversation': General chat, questions, or information
- 'action': Execute a specific operation or task"""
    
    result = structured_llm.invoke(prompt)
    
    if result.intent == "action":
        return "action"
    return "__end__"
```

---

## Orchestrator Agent

Multi-agent coordinator pattern.

### File: src/agents/orchestrator.py

```python
"""
Orchestrator agent that coordinates multiple sub-agents.
"""

from typing import List, Literal

from langchain.chat_models import init_chat_model
from langchain_core.messages import AIMessage
from langgraph.graph import StateGraph, START, END, MessagesState
from langgraph.types import Send
from pydantic import BaseModel, Field


class Task(BaseModel):
    """A task to be executed by a sub-agent."""
    agent: Literal["researcher", "writer", "reviewer"]
    instruction: str


class Plan(BaseModel):
    """Orchestrator's execution plan."""
    tasks: List[Task] = Field(description="List of tasks to execute")


class State(MessagesState):
    """Orchestrator state."""
    plan: Plan
    results: List[str]


llm = init_chat_model("openai:gpt-4o", temperature=0)
planner_llm = llm.with_structured_output(schema=Plan)


def orchestrator(state: State) -> dict:
    """Create execution plan for sub-agents."""
    messages = state["messages"]
    
    prompt = """Analyze the user request and create a plan.
    Available agents: researcher, writer, reviewer.
    Break down the task into steps for each agent."""
    
    plan = planner_llm.invoke([*messages, {"role": "system", "content": prompt}])
    return {"plan": plan, "results": []}


def route_tasks(state: State) -> List[Send]:
    """Route tasks to appropriate sub-agents."""
    sends = []
    for task in state["plan"].tasks:
        sends.append(Send(task.agent, {"task": task.instruction}))
    return sends


def researcher(state: dict) -> dict:
    """Research agent."""
    task = state["task"]
    response = llm.invoke(f"Research: {task}")
    return {"results": [f"Research: {response.content}"]}


def writer(state: dict) -> dict:
    """Writer agent."""
    task = state["task"]
    response = llm.invoke(f"Write: {task}")
    return {"results": [f"Writing: {response.content}"]}


def reviewer(state: dict) -> dict:
    """Reviewer agent."""
    task = state["task"]
    response = llm.invoke(f"Review: {task}")
    return {"results": [f"Review: {response.content}"]}


def aggregator(state: State) -> dict:
    """Aggregate results from all sub-agents."""
    results = state.get("results", [])
    summary = "\n".join(results)
    return {"messages": [AIMessage(content=f"Completed tasks:\n{summary}")]}


# Graph construction
builder = StateGraph(State)
builder.add_node("orchestrator", orchestrator)
builder.add_node("researcher", researcher)
builder.add_node("writer", writer)
builder.add_node("reviewer", reviewer)
builder.add_node("aggregator", aggregator)

builder.add_edge(START, "orchestrator")
builder.add_conditional_edges("orchestrator", route_tasks)
builder.add_edge("researcher", "aggregator")
builder.add_edge("writer", "aggregator")
builder.add_edge("reviewer", "aggregator")
builder.add_edge("aggregator", END)

agent = builder.compile()
```

---

## Evaluator Agent

Generator + Evaluator iterative pattern.

### File: src/agents/evaluator.py

```python
"""
Evaluator agent with generation and evaluation loop.
"""

from typing import Literal

from langchain.chat_models import init_chat_model
from langchain_core.messages import AIMessage
from langgraph.graph import StateGraph, START, END, MessagesState
from pydantic import BaseModel, Field


class Evaluation(BaseModel):
    """Evaluation result."""
    score: int = Field(description="Quality score 1-10")
    feedback: str = Field(description="Improvement suggestions")
    passed: bool = Field(description="Whether the output passes quality threshold")


class State(MessagesState):
    """Evaluator agent state."""
    generation: str
    evaluation: Evaluation
    attempts: int


llm = init_chat_model("openai:gpt-4o", temperature=0.7)
evaluator_llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_evaluator = evaluator_llm.with_structured_output(schema=Evaluation)


def generator(state: State) -> dict:
    """Generate content based on user request."""
    messages = state["messages"]
    
    # Include feedback if this is a retry
    if state.get("evaluation") and not state["evaluation"].passed:
        feedback = state["evaluation"].feedback
        messages = [*messages, {"role": "system", "content": f"Previous feedback: {feedback}"}]
    
    response = llm.invoke(messages)
    attempts = state.get("attempts", 0) + 1
    
    return {
        "generation": response.content,
        "attempts": attempts,
    }


def evaluator(state: State) -> dict:
    """Evaluate generated content."""
    generation = state["generation"]
    original_request = state["messages"][0].content
    
    prompt = f"""Evaluate this generated content:

Original request: {original_request}

Generated content:
{generation}

Score from 1-10 and provide feedback. Pass if score >= 7."""
    
    evaluation = structured_evaluator.invoke(prompt)
    return {"evaluation": evaluation}


def should_retry(state: State) -> Literal["generator", "output"]:
    """Decide whether to retry generation."""
    MAX_ATTEMPTS = 3
    
    if state["evaluation"].passed:
        return "output"
    
    if state["attempts"] >= MAX_ATTEMPTS:
        return "output"
    
    return "generator"


def output(state: State) -> dict:
    """Output final generation."""
    return {
        "messages": [AIMessage(content=state["generation"])]
    }


# Graph construction
builder = StateGraph(State)
builder.add_node("generator", generator)
builder.add_node("evaluator", evaluator)
builder.add_node("output", output)

builder.add_edge(START, "generator")
builder.add_edge("generator", "evaluator")
builder.add_conditional_edges("evaluator", should_retry)
builder.add_edge("output", END)

agent = builder.compile()
```
