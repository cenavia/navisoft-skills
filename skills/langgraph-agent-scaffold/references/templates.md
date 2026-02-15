# Templates Reference

## Table of Contents
- [Configuration Files](#configuration-files)
- [Development Tools](#development-tools)
- [Source Files](#source-files)
- [API Files](#api-files)
- [Notebook Template](#notebook-template)

---

## Configuration Files

### .python-version
```
3.12
```

### .env.example
```bash
# OpenAI Configuration
OPENAI_API_KEY=your-openai-api-key

# Anthropic Configuration (optional)
ANTHROPIC_API_KEY=your-anthropic-api-key

# LangSmith Configuration (optional)
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your-langsmith-api-key
LANGCHAIN_PROJECT={project_name}

# Database Configuration
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/{project_name}
```

### pyproject.toml
```toml
[project]
name = "{project_name}"
version = "0.1.0"
description = "AI Agent built with LangGraph"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "langgraph>=1.0.3",
    "langchain>=1.0.5",
    "langchain-openai>=1.0.2",
    "langchain-anthropic>=1.0.2",
    "langgraph-checkpoint-postgres>=1.0.0",
    "fastapi>=0.117.1",
    "uvicorn>=0.34.0",
    "jinja2>=3.1.6",
    "pydantic>=2.10.0",
    "python-dotenv>=1.0.0",
    "psycopg[binary]>=3.2.0",
]

[dependency-groups]
dev = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.24.0",
    "ipykernel>=6.29.0",
    "langgraph-cli[inmem]>=0.4.11",
    "ruff>=0.8.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src"]

[tool.ruff]
line-length = 88
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W"]
```

---

## Development Tools

### justfile
```makefile
# Default recipe
default:
    @just --list

# Install dependencies
install:
    uv sync

# Install with dev dependencies
install-dev:
    uv sync --group dev

# Run LangGraph in development mode
run:
    uv run langgraph dev

# Start PostgreSQL with Docker
db-up:
    docker-compose up -d

# Stop PostgreSQL
db-down:
    docker-compose down

# Run tests
test:
    uv run pytest

# Format code
format:
    uv run ruff format src/

# Lint code
lint:
    uv run ruff check src/

# Lint and fix
lint-fix:
    uv run ruff check src/ --fix
```

### docker-compose.yml
```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: {project_name}-db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: {project_name}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

### langgraph.json
```json
{
  "$schema": "https://langchain-ai.github.io/langgraph/schemas/langgraph.json",
  "dependencies": ["."],
  "graphs": {
    "agent": "./src/agents/main.py:agent"
  },
  "env": ".env"
}
```

---

## Source Files

### src/__init__.py
```python
"""
{project_name} - AI Agent built with LangGraph
"""
```

### src/agents/__init__.py
```python
"""
Agent definitions for {project_name}
"""

from agents.main import agent

__all__ = ["agent"]
```

---

## API Files

### src/api/__init__.py
```python
"""
FastAPI application for {project_name}
"""
```

### src/api/main.py
```python
"""
FastAPI endpoints for the AI agent.
"""

from contextlib import asynccontextmanager
from typing import Annotated

from fastapi import FastAPI, Depends
from pydantic import BaseModel

from api.db import get_checkpointer
from agents import agent


class ChatRequest(BaseModel):
    """Request model for chat endpoint."""
    message: str
    thread_id: str


class ChatResponse(BaseModel):
    """Response model for chat endpoint."""
    response: str
    thread_id: str


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Application lifespan handler."""
    yield


app = FastAPI(
    title="{project_name}",
    description="AI Agent API",
    version="0.1.0",
    lifespan=lifespan,
)


@app.get("/health")
async def health_check() -> dict:
    """Health check endpoint."""
    return {"status": "healthy"}


@app.post("/chat", response_model=ChatResponse)
async def chat(request: ChatRequest) -> ChatResponse:
    """
    Chat with the AI agent.
    
    Args:
        request: Chat request with message and thread_id
        
    Returns:
        ChatResponse with agent's response
    """
    config = {"configurable": {"thread_id": request.thread_id}}
    
    result = agent.invoke(
        {"messages": [{"role": "user", "content": request.message}]},
        config=config,
    )
    
    last_message = result["messages"][-1]
    
    return ChatResponse(
        response=last_message.content,
        thread_id=request.thread_id,
    )
```

### src/api/db.py
```python
"""
Database configuration and checkpointer setup.
"""

import os
from functools import lru_cache

from dotenv import load_dotenv
from langgraph.checkpoint.postgres import PostgresSaver

load_dotenv()


@lru_cache
def get_database_url() -> str:
    """Get database URL from environment."""
    return os.getenv(
        "DATABASE_URL",
        "postgresql://postgres:postgres@localhost:5432/{project_name}"
    )


def get_checkpointer() -> PostgresSaver:
    """
    Create and return a PostgreSQL checkpointer.
    
    Returns:
        PostgresSaver: Configured checkpointer instance
    """
    return PostgresSaver.from_conn_string(get_database_url())
```

---

## Notebook Template

### notebooks/01-getting-started.ipynb

Create a Jupyter notebook with the following cells:

**Cell 1 (Markdown):**
```markdown
# {project_name} - Getting Started

This notebook demonstrates how to use the AI agent.

## Setup

Make sure you have:
1. Installed dependencies: `just install`
2. Set up environment variables in `.env`
3. Started PostgreSQL: `just db-up`
```

**Cell 2 (Code):**
```python
import os
from dotenv import load_dotenv

load_dotenv()

# Verify API key is set
assert os.getenv("OPENAI_API_KEY"), "OPENAI_API_KEY not set in .env"
print("✅ Environment configured")
```

**Cell 3 (Markdown):**
```markdown
## Import the Agent
```

**Cell 4 (Code):**
```python
from agents import agent

# Display the agent graph
agent.get_graph().print_ascii()
```

**Cell 5 (Markdown):**
```markdown
## Test the Agent
```

**Cell 6 (Code):**
```python
from langchain_core.messages import HumanMessage

# Invoke the agent
result = agent.invoke({
    "messages": [HumanMessage(content="Hello! What can you help me with?")]
})

# Display response
print(result["messages"][-1].content)
```
