# README Template

Use this template to generate the README.md for each scaffolded project.

---

# {project_name}

AI Agent built with LangGraph and LangChain.

## Overview

This project implements an AI agent using the LangGraph framework with the following capabilities:

- **State-based graph architecture** for predictable agent behavior
- **Tool integration** for extending agent capabilities
- **Checkpoint persistence** with PostgreSQL for conversation memory
- **FastAPI endpoints** for REST API access
- **LangGraph Studio** support for visual development

## Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| Python | >=3.12 | Runtime |
| LangGraph | >=1.0.3 | Agent framework |
| LangChain | >=1.0.5 | LLM orchestration |
| FastAPI | >=0.117.1 | REST API |
| PostgreSQL | 16 | Checkpoint storage |
| UV | latest | Package management |

## Project Structure

```
{project_name}/
├── src/
│   ├── agents/           # Agent definitions
│   │   ├── main.py       # Main agent
│   │   └── {agent_type}/ # Modular agent (if applicable)
│   │       ├── agent.py  # Graph definition
│   │       ├── state.py  # State class
│   │       ├── nodes/    # Node implementations
│   │       └── routes/   # Conditional routing
│   └── api/              # FastAPI application
│       ├── main.py       # API endpoints
│       └── db.py         # Database configuration
├── notebooks/            # Jupyter notebooks
├── pyproject.toml        # Project configuration
├── langgraph.json        # LangGraph Studio config
├── docker-compose.yml    # PostgreSQL setup
└── justfile              # Development commands
```

## Quick Start

### Prerequisites

- Python 3.12+
- UV package manager
- Docker (for PostgreSQL)
- OpenAI API key

### Installation

1. **Clone and enter the project:**
   ```bash
   cd {project_name}
   ```

2. **Install dependencies with UV:**
   ```bash
   uv sync
   ```

   For development dependencies:
   ```bash
   uv sync --group dev
   ```

3. **Configure environment variables:**
   ```bash
   cp .env.example .env
   # Edit .env with your API keys
   ```

4. **Start PostgreSQL:**
   ```bash
   just db-up
   ```

5. **Run in development mode:**
   ```bash
   uv run langgraph dev
   ```

## Development Commands

| Command | Description |
|---------|-------------|
| `uv sync` | Install dependencies |
| `uv sync --group dev` | Install with dev dependencies |
| `uv run langgraph dev` | Start LangGraph development server |
| `just db-up` | Start PostgreSQL container |
| `just db-down` | Stop PostgreSQL container |
| `uv run pytest` | Run tests |
| `uv run ruff format src/` | Format code with Ruff |
| `uv run ruff check src/` | Lint code with Ruff |

## Architecture Reference

### State Management

All agent states inherit from `MessagesState`:

```python
from langgraph.graph import MessagesState

class State(MessagesState):
    customer_name: str
    additional_field: dict
```

### Node Pattern

Nodes receive state and return partial updates:

```python
def my_node(state: State) -> dict:
    # Process state
    return {"field": new_value, "messages": [response]}
```

### Conditional Routing

Routes return the next node name:

```python
from typing import Literal

def my_route(state: State) -> Literal["node_a", "node_b", "__end__"]:
    if condition:
        return "node_a"
    return "__end__"
```

### Tool Definition

Use the `@tool` decorator with descriptions:

```python
from langchain_core.tools import tool

@tool("tool_name", description="What this tool does")
def my_tool(param: str) -> str:
    """Detailed docstring for the tool."""
    return result
```

### Prompt Templates

Use Jinja2 for dynamic prompts:

```python
from langchain_core.prompts import PromptTemplate

template = """\
{% if name %}Hello {{ name }}!{% endif %}
Task: {{ task }}
"""
prompt = PromptTemplate.from_template(template, template_format="jinja2")
```

## Code Conventions

### File Organization

| Location | Purpose |
|----------|---------|
| `agents/{name}/agent.py` | Graph definition |
| `agents/{name}/state.py` | State class |
| `agents/{name}/nodes/{node}/node.py` | Node logic |
| `agents/{name}/nodes/{node}/prompt.py` | Prompts |
| `agents/{name}/nodes/{node}/tools.py` | Tools |
| `agents/{name}/routes/{route}/route.py` | Routing logic |

### Import Order

```python
# 1. Built-in
from typing import TypedDict, Literal

# 2. External - LangGraph
from langgraph.graph import StateGraph, START, END

# 3. External - LangChain
from langchain.chat_models import init_chat_model

# 4. External - Others
from pydantic import BaseModel, Field

# 5. Internal
from agents.support.state import State
```

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Files/Folders | snake_case | `booking_node.py` |
| Classes | PascalCase | `BookingState` |
| Functions | snake_case | `process_booking` |
| Constants | UPPER_SNAKE_CASE | `SYSTEM_PROMPT` |
| Node functions | snake_case | `conversation` |
| Route functions | snake_case + `_route` | `intent_route` |

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | Yes | OpenAI API key |
| `ANTHROPIC_API_KEY` | No | Anthropic API key |
| `DATABASE_URL` | No | PostgreSQL connection string |
| `LANGCHAIN_API_KEY` | No | LangSmith API key |
| `LANGCHAIN_TRACING_V2` | No | Enable LangSmith tracing |

### LangGraph Studio

Configure agents in `langgraph.json`:

```json
{
  "graphs": {
    "agent": "./src/agents/main.py:agent"
  }
}
```

Run with: `uv run langgraph dev`

## Testing

Run tests with pytest:

```bash
just test
```

Use notebooks for interactive development:

```bash
jupyter lab notebooks/
```

## Extending the Project

### Adding a New Node

1. Create folder: `src/agents/{agent}/nodes/{node_name}/`
2. Add files: `__init__.py`, `node.py`, `prompt.py`, `tools.py` (if needed)
3. Implement node function in `node.py`
4. Register in `agent.py` with `builder.add_node()`

### Adding a New Route

1. Create folder: `src/agents/{agent}/routes/{route_name}/`
2. Add files: `__init__.py`, `route.py`
3. Implement route function returning `Literal` types
4. Register with `builder.add_conditional_edges()`

### Adding a New Tool

1. Add to appropriate `tools.py` file
2. Use `@tool` decorator with description
3. Bind to LLM with `llm.bind_tools([tool])`

## License

[Add your license here]
