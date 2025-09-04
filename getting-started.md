---
title: Getting Started
nav_order: 2
---

# Getting Started

This guide will help you get up and running with the ADK template in under 10 minutes.

## Prerequisites

Before you begin, ensure you have:
- Python 3.11 or higher installed
- A code editor (VS Code, PyCharm, etc.)
- Basic Python knowledge
- (Optional) Google Cloud account for advanced features

**If you are using Replit, go straight to Step 2**

## Step 1: Environment Setup

### Option A: Using UV (Recommended)

UV is a fast Python package manager that handles dependencies efficiently.

```bash
# Install UV if you haven't already
curl -LsSf https://astral.sh/uv/install.sh | sh

# Clone the repository
git clone <your-repo-url>
cd SummitTemplate

# Install dependencies
uv sync
```

### Option B: Using pip

```bash
# Clone the repository
git clone <your-repo-url>
cd SummitTemplate

# Create a virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install google-adk jupyter
```

## Step 2: Understanding the Project Structure

```
SummitTemplate/
├── demo_agent/
│   ├── agent.py        # Your main agent definition
│   ├── prompt.py       # System prompts and instructions
│   └── __init__.py     # Package initialization
├── docs/               # Documentation (you are here!)
├── pyproject.toml      # Project configuration
└── .replit            # Replit deployment config
```

### Key Files Explained

**`demo_agent/agent.py`** - The heart of your agent:
```python
# Core imports
from google.adk.agents import Agent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset

# Agent definition
root_agent = Agent(
    name="assistant_chatbot",
    model="gemini-2.5-flash",
    description="An assistant agent that helps users",
    instruction=SYSTEM_PROMPT
)
```

**`demo_agent/prompt.py`** - Agent instructions:
```python
SYSTEM_PROMPT = "You are a helpful assistant who can help with online search tasks..."
```

## Step 3: Running Your First Agent

### Local Development Server

The ADK provides a built-in web interface for testing agents:

```bash
# Start the development server
adk web
```

- A chat interface to interact with your agent
- Real-time tool execution logs
- Agent response streaming

**On Replit, click the Run button to open the development UI**

### Testing the Demo Agent

1. Start the server (as shown above)
2. In the chat interface, try these prompts:
   - "What's the current time?"
   - "Search for the latest news about AI"
   - "Help me find information about Python programming"

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
- Additional tools that can be dynamically loaded

### Using Your Own MCP Server

If you have your own MCP server:

```python
def get_mcp_toolset(user_id: str = 'guest_user') -> MCPToolset:
    url = 'YOUR_MCP_SERVER_URL'  # Replace with your server
    # ... rest of the configuration
```

## Step 6: Testing and Debugging

### Enable Logging

Add logging to understand what your agent is doing:

```python
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Use in your code
logger.info(f"Processing request: {user_input}")
logger.debug(f"Tool response: {response}")
```

### Hot Reload for Development

The `--reload_agents` flag enables hot reloading:
```bash
adk web --reload_agents --port 5000
```

Now when you modify your agent code, it automatically reloads!

## Step 7: Next Steps

### Immediate Next Steps

1. ✅ Modify the agent name and description for your project
2. ✅ Update the system prompt to match your use case
3. ✅ Test the agent with relevant prompts
4. ✅ Add one custom tool specific to your needs

### Learn More

- Read the [First Agent Tutorial](./tutorials/01-first-agent.md) for detailed explanations
- Explore [Working with Tools](./tutorials/02-working-with-tools.md) to add capabilities
- Check out [MCP Integration Guide](./tutorials/03-mcp-integration.md) for advanced features

### Deployment Options

When ready to deploy:
- **Replit**: Use the included `.replit` configuration
- **Cloud Run**: Deploy as a containerized service
- **Vertex AI**: Use Google's managed AI platform

## Troubleshooting

### Common Issues and Solutions

**Port already in use:**
```bash
# Use a different port
adk web --reload_agents --port 5001
```

**Module not found errors:**
```bash
# Ensure you're in the virtual environment
source venv/bin/activate  # or use UV
uv sync
```

**MCP server connection issues:**
- Check your internet connection
- Verify the server URL is correct
- Check if you need authentication tokens

**Agent not responding:**
- Check the system prompt isn't too restrictive
- Verify tools are properly imported
- Look at the console logs for errors

## Getting Help

- Check the [official ADK documentation](https://google.github.io/adk-docs/)
- Review the example code in `demo_agent/`
- Look at the [Examples](./examples/) directory for more patterns

---

Ready to build? Head to the [tutorials](./tutorials/) to dive deeper! 🚀