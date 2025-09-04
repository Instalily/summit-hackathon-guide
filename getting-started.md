---
title: Getting Started
layout: default
nav_order: 2
---

# Getting Started

This guide walks you through setting up the Agent Development Kit (ADK) and running your first agent in under 10 minutes.

## Prerequisites

Before you begin, ensure you have:

| Requirement | Details |
|-------------|----------|
| **Python** | Version 3.11 or higher |
| **Editor** | VS Code, PyCharm, or any code editor |
| **Knowledge** | Basic Python understanding |
| **Optional** | Google Cloud account for advanced features |

**Note:** **If using Replit, skip to Step 2.**

## Step 1: Environment Setup

### Option A: From Local Repository (If you downloaded the template)

If you have the repository with `pyproject.toml`:

```bash
# Navigate to your downloaded repository
cd path/to/your/AgentQuickstart

# Using UV (Fastest - recommended)
curl -LsSf https://astral.sh/uv/install.sh | sh  # Install UV first
uv sync  # This reads pyproject.toml and installs everything

# OR using pip with pyproject.toml
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -e .  # Installs in editable mode from pyproject.toml
```

## Step 2: Understanding the Project Structure

```
SummitTemplate/
├── demo_agent/
│   ├── agent.py        # Your main agent definition
│   ├── prompt.py       # System prompts and instructions
│   └── __init__.py     # Package initialization
```

### Key Files

#### `demo_agent/agent.py`

```python
# Core imports
from google.adk.agents import Agent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset

# Your main agent definition
root_agent = Agent(
    name="assistant_chatbot",       # Unique identifier
    model="gemini-2.5-flash",        # LLM to use
    description="An assistant agent that helps users",
    instruction=SYSTEM_PROMPT        # Behavioral instructions
)
```

#### `demo_agent/prompt.py`

```python
SYSTEM_PROMPT = """
You are a helpful assistant who can:
- Search the web for information
- Answer questions accurately
- Provide helpful suggestions
Always be concise and friendly!
"""
```

## Step 3: Running Your First Agent

### Launch the Development Server

```bash
# Start the ADK web interface
adk web

# The server will start and display within the terminal. CMD click to open.
```

**Replit Users**: Click the Run button.

### Interface Features

- Chat interface for agent interaction
- Tool execution logs
- Real-time response streaming
- Debug panel for troubleshooting

### Test Prompts

Try these examples:
- "What's the current time?"
- "Search for the latest news about AI"
- "Help me understand recursion in programming"

## Step 4: Customizing Your Agent

### Modify the Agent Name and Model

Edit `demo_agent/agent.py`:

```python
root_agent = Agent(
    name="my_hackathon_agent",  # Change this
    model="gemini-2.5-flash",    # Or use "gemini-1.5-pro", "gpt-4", etc.
    description="A specialized agent for [your use case]",
    instruction="You are an expert in [domain]. Help users with [tasks]...",
    tools=[...]
)
```

### Update the System Prompt

Edit `demo_agent/prompt.py`:

```python
SYSTEM_PROMPT = """
You are a specialized assistant for [your hackathon project].
Your capabilities include:
- [Capability 1]
- [Capability 2]
- [Capability 3]

Always:
- Be helpful and accurate
- Cite sources when providing information
- Ask for clarification when needed
"""
```

### Add a Custom Tool

Add a new function in `demo_agent/agent.py`:

```python
def analyze_data(data: str) -> dict:
    """Analyzes provided data and returns insights.
    
    Args:
        data: The data to analyze
        
    Returns:
        dict: Analysis results
    """
    # Your analysis logic here
    result = process_data(data)
    return {"status": "success", "analysis": result}

# Add to agent tools
root_agent = Agent(
    ...
    tools=[get_mcp_toolset(), get_current_time, analyze_data]
)
```

## Step 5: Working with MCP Server

The template includes integration with an MCP (Model Context Protocol) server that provides additional capabilities.

### Understanding MCP Integration

```python
def get_mcp_toolset(user_id: str = 'guest_user') -> MCPToolset:
    """Gets tools from the MCP Server."""
    url = 'https://summit-mcp-server-586731320826.us-central1.run.app'
    
    headers = {"X-User-Id": user_id}  # User identification
    
    toolset = MCPToolset(
        connection_params=SseConnectionParams(
            url=f"{url}/sse", 
            headers=headers
        )
    )
    return toolset
```

The MCP server provides:
- Online search capabilities
- Quick answers with citations