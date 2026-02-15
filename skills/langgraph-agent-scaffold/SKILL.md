---
name: langgraph-agent-scaffold
description: Generate complete project scaffolds for AI agents using LangGraph and LangChain. Use this skill when users need to create a new AI agent project from scratch, initialize a LangGraph-based project structure, or scaffold multi-agent systems with proper architecture. Triggers include requests to create agent projects, initialize LangGraph workspaces, build RAG agents, create orchestrator patterns, or set up support agent structures.
---

# LangGraph Agent Scaffold Generator

Generate production-ready scaffolds for AI agent projects using LangGraph and LangChain.

## Workflow

### Step 1: Gather Project Information

**Required input from user:**
- `project_name`: Name of the project (kebab-case, e.g., `my-agent-project`)

**Optional inputs:**
- `agent_type`: Type of agent to scaffold (default: `simple`)
  - `simple` - Basic agent with tools
  - `react` - ReAct reasoning agent
  - `rag` - Retrieval-Augmented Generation agent
  - `modular` - Modular agent with nodes/routes structure (customizable for any domain)
  - `orchestrator` - Multi-agent coordinator
  - `evaluator` - Generator + Evaluator pattern

If project name not provided, ask: "What name would you like for your agent project? (use kebab-case, e.g., my-agent-project)"

### Step 2: Generate Project Structure

Create the following directory structure:

```
{project_name}/
├── .env.example
├── .python-version
├── pyproject.toml
├── langgraph.json
├── docker-compose.yml
├── justfile
├── README.md
├── src/
│   ├── __init__.py
│   ├── agents/
│   │   ├── __init__.py
│   │   └── {agent files based on type}
│   └── api/
│       ├── __init__.py
│       ├── main.py
│       └── db.py
└── notebooks/
    └── 01-getting-started.ipynb
```

### Step 3: Generate Files

Use templates from [references/templates.md](references/templates.md) for file contents.

For agent-specific patterns, consult [references/agent-patterns.md](references/agent-patterns.md).

### Step 4: Generate README.md

Create comprehensive documentation following the template in [references/readme-template.md](references/readme-template.md).

## File Generation Order

1. Configuration files (`.python-version`, `pyproject.toml`, `.env.example`)
2. Development tools (`justfile`, `docker-compose.yml`, `langgraph.json`)
3. Source structure (`src/__init__.py`, `src/agents/__init__.py`, `src/api/__init__.py`)
4. Agent files (based on selected type)
5. API files (`main.py`, `db.py`)
6. Notebook template
7. README.md

## Code Style Rules

### Python Standards
- Python 3.12+ required
- Type hints mandatory on all functions
- Docstrings required for public methods
- PEP 8 naming: `snake_case` functions/variables, `PascalCase` classes

### Import Order
```python
# Built-in
from typing import TypedDict, Literal

# External - LangGraph
from langgraph.graph import StateGraph, START, END

# External - LangChain
from langchain.chat_models import init_chat_model

# External - Others
from pydantic import BaseModel, Field

# Internal
from agents.support.state import State
```

### State Definition
Always inherit from `MessagesState`:
```python
from langgraph.graph import MessagesState

class State(MessagesState):
    customer_name: str
    extracted_data: dict
```

### Node Functions
Return partial state dictionaries:
```python
def my_node(state: State) -> dict:
    return {"messages": [ai_message], "field": value}
```

### Tools
Use decorator with description:
```python
@tool("tool_name", description="Clear description")
def my_tool(param: str) -> str:
    """Detailed docstring."""
    return result
```

### Prompts
Use Jinja2 templates:
```python
template = """\
{% if name %}Hello {{ name }}{% endif %}
"""
prompt = PromptTemplate.from_template(template, template_format="jinja2")
```

## Validation Checklist

After generation, verify:
- [ ] All `__init__.py` files created
- [ ] Type hints on all function signatures
- [ ] Proper import organization
- [ ] States inherit from `MessagesState`
- [ ] Nodes return dictionaries
- [ ] Tools have descriptions and docstrings
- [ ] `.env.example` contains required variables
- [ ] README.md includes setup instructions
