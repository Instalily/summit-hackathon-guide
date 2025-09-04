---
title: First Agent
layout: default
nav_order: 3
---

# Building Your First Agent

In this tutorial, you'll build a complete AI agent from scratch using Google's Agent Development Kit (ADK).

## What You'll Build

A Research Assistant agent that can:
- Answer questions using LLMs
- Search the web for information
- Provide time across different cities
- Perform calculations
- Cite sources

## Prerequisites

- Completed [Getting Started](getting-started)
- Python 3.11+ environment
- ADK installed (`pip install google-adk`)
- Basic Python knowledge

## Part 1: Understanding Agent Components

### Core Components

| Component | Purpose | Example |
|-----------|---------|----------|
| **Agent Definition** | Identity & configuration | Name, description, model |
| **Model Selection** | LLM choice | Gemini 2.5, GPT-4, Claude |
| **Instructions** | System prompts | Behavior guidelines |
| **Tools** | Functions | Search, calculate, fetch data |

## Part 2: Creating a Basic Agent

### Step 1: Create Your Agent File

Create a new file called `my_first_agent.py`:

```python
from google.adk.agents import Agent

# Define a simple agent
simple_agent = Agent(
    name="simple_assistant",
    model="gemini-2.5-flash",
    description="A simple helpful assistant",
    instruction="You are a helpful assistant. Be concise and friendly."
)

# This makes the agent available to ADK
root_agent = simple_agent
```

### Step 2: Launch Your Agent

```bash
# Save the file and start the server
adk web

# Server will start at http://localhost:5000
```

### Step 3: Test Your Agent

Try these prompts:
- "Hello! Who are you?"
- "What can you help me with?"
- "Explain quantum computing in simple terms"

## Part 3: Adding Custom Tools

Tools are Python functions that give your agent real-world capabilities.

### Step 1: Create a Time Tool

Update your `my_first_agent.py`:

```python
from google.adk.agents import Agent
import datetime
from zoneinfo import ZoneInfo

def get_current_time(city: str = "New York") -> dict:
    """Get the current time in a specified city.
    
    Args:
        city: Name of the city (default: New York)
        
    Returns:
        dict: Contains status and time information
    """
    # Map cities to timezones
    timezone_map = {
        "New York": "America/New_York",
        "London": "Europe/London",
        "Tokyo": "Asia/Tokyo",
        "Sydney": "Australia/Sydney",
        "Chicago": "America/Chicago",
        "Los Angeles": "America/Los_Angeles"
    }
    
    if city not in timezone_map:
        return {
            "status": "error",
            "message": f"Unknown city: {city}. Supported cities: {', '.join(timezone_map.keys())}"
        }
    
    tz = ZoneInfo(timezone_map[city])
    now = datetime.datetime.now(tz)
    
    return {
        "status": "success",
        "city": city,
        "time": now.strftime("%I:%M %p"),
        "date": now.strftime("%B %d, %Y"),
        "timezone": timezone_map[city]
    }

# Update agent with tool
simple_agent = Agent(
    name="simple_assistant",
    model="gemini-2.5-flash",
    description="A helpful assistant with time capabilities",
    instruction="""You are a helpful assistant. Be concise and friendly.
    When asked about time, use the get_current_time tool.""",
    tools=[get_current_time]  # Add the tool here
)

root_agent = simple_agent
```

### Step 2: Test the Tool

```bash
# Restart the server
adk web --reload_agents
```

Test with:
- "What time is it in Tokyo?"
- "Tell me the current time in London and New York"
- "What's the date in Sydney?"

## Part 4: Adding a Calculator Tool

Let's add mathematical capabilities:

```python
import math

def calculator(expression: str, operation: str = "evaluate") -> dict:
    """Perform mathematical calculations.
    
    Args:
        expression: Mathematical expression to evaluate
        operation: Type of operation (evaluate, sqrt, factorial)
        
    Returns:
        dict: Calculation result
    """
    try:
        if operation == "evaluate":
            # Safe evaluation of mathematical expressions
            allowed_names = {
                k: v for k, v in math.__dict__.items() if not k.startswith("__")
            }
            result = eval(expression, {"__builtins__": {}}, allowed_names)
        elif operation == "sqrt":
            result = math.sqrt(float(expression))
        elif operation == "factorial":
            result = math.factorial(int(expression))
        else:
            return {"status": "error", "message": f"Unknown operation: {operation}"}
            
        return {
            "status": "success",
            "expression": expression,
            "operation": operation,
            "result": result
        }
    except Exception as e:
        return {
            "status": "error",
            "message": f"Calculation error: {str(e)}"
        }

# Add to agent tools
simple_agent = Agent(
    name="simple_assistant",
    model="gemini-2.5-flash",
    description="A helpful assistant with time and math capabilities",
    instruction="""You are a helpful assistant. Be concise and friendly.
    Use the available tools when appropriate:
    - get_current_time for time-related queries
    - calculator for mathematical computations""",
    tools=[get_current_time, calculator]
)
```

Test with:
- "What's 25 * 4?"
- "Calculate the square root of 144"
- "What's 5 factorial?"

## Part 5: Improving Prompts

### Step 1: Create a Prompts File

Create `prompts.py`:

```python
RESEARCH_ASSISTANT_PROMPT = """
You are a knowledgeable research assistant designed to help users find and understand information.

Your core capabilities:
1. Answering questions accurately and concisely
2. Performing calculations when needed
3. Providing current time information for major cities
4. Breaking down complex topics into understandable explanations

Guidelines for your responses:
- Always be accurate and cite your reasoning
- If you're unsure about something, say so
- Use tools when they would provide more accurate information
- Keep responses concise unless asked for detail
- Be friendly and professional

When using tools:
- Use the calculator for any mathematical computations
- Use get_current_time for time-related queries
- Explain what you're doing when you use a tool

Remember: Your goal is to be helpful, accurate, and easy to understand.
"""
```

### Step 2: Update Your Agent

```python
from prompts import RESEARCH_ASSISTANT_PROMPT

research_agent = Agent(
    name="research_assistant",
    model="gemini-2.5-flash",
    description="A knowledgeable research assistant",
    instruction=RESEARCH_ASSISTANT_PROMPT,
    tools=[get_current_time, calculator]
)

root_agent = research_agent
```

## Part 6: Adding Web Search with MCP

Now let's integrate the MCP server for web search capabilities.

```python
from google.adk.agents import Agent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset, SseConnectionParams
from prompts import RESEARCH_ASSISTANT_PROMPT
import logging

logger = logging.getLogger(__name__)

def get_mcp_tools() -> MCPToolset:
    """Connect to MCP server for additional tools."""
    mcp_url = 'https://summit-mcp-server-586731320826.us-central1.run.app'
    
    try:
        toolset = MCPToolset(
            connection_params=SseConnectionParams(
                url=f"{mcp_url}/sse",
                headers={"X-User-Id": "hackathon_user"}
            )
        )
        logger.info("Successfully connected to MCP server")
        return toolset
    except Exception as e:
        logger.error(f"Failed to connect to MCP server: {e}")
        return None

# Create agent with all tools
research_agent = Agent(
    name="research_assistant",
    model="gemini-2.5-flash",
    description="A research assistant with web search capabilities",
    instruction=RESEARCH_ASSISTANT_PROMPT + """
    
    You also have access to web search tools through MCP. Use these for:
    - Current events and news
    - Factual information you're unsure about
    - Recent developments in any field
    
    Always cite sources when using web search results.""",
    tools=[get_mcp_tools(), get_current_time, calculator]
)

root_agent = research_agent
```

## Part 7: Complete Example

Here's the final, complete agent:

```python
# my_research_agent.py
from google.adk.agents import Agent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset, SseConnectionParams
import datetime
from zoneinfo import ZoneInfo
import math
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# System Prompt
SYSTEM_PROMPT = """
You are a knowledgeable research assistant designed to help users find and understand information.

Your core capabilities:
1. Web search for current information
2. Mathematical calculations
3. Time information for major cities
4. Clear explanations of complex topics

Guidelines:
- Be accurate and cite sources
- Use tools when they provide better information
- Keep responses concise unless asked for detail
- Be friendly and professional
- Explain your reasoning when appropriate
"""

def get_current_time(city: str = "New York") -> dict:
    """Get current time in a city."""
    timezone_map = {
        "New York": "America/New_York",
        "London": "Europe/London",
        "Tokyo": "Asia/Tokyo",
        "Sydney": "Australia/Sydney",
        "Chicago": "America/Chicago",
        "Los Angeles": "America/Los_Angeles",
        "Paris": "Europe/Paris",
        "Dubai": "Asia/Dubai",
        "Singapore": "Asia/Singapore"
    }
    
    if city not in timezone_map:
        return {"status": "error", "message": f"Unknown city. Supported: {', '.join(timezone_map.keys())}"}
    
    tz = ZoneInfo(timezone_map[city])
    now = datetime.datetime.now(tz)
    
    return {
        "status": "success",
        "city": city,
        "time": now.strftime("%I:%M %p"),
        "date": now.strftime("%B %d, %Y"),
        "day": now.strftime("%A"),
        "timezone": timezone_map[city]
    }

def calculator(expression: str) -> dict:
    """Evaluate mathematical expressions."""
    try:
        # Safe math evaluation
        allowed_names = {k: v for k, v in math.__dict__.items() if not k.startswith("__")}
        result = eval(expression, {"__builtins__": {}}, allowed_names)
        return {"status": "success", "expression": expression, "result": result}
    except Exception as e:
        return {"status": "error", "message": str(e)}

def get_mcp_toolset() -> MCPToolset:
    """Connect to MCP server for web search."""
    url = 'https://summit-mcp-server-586731320826.us-central1.run.app'
    return MCPToolset(
        connection_params=SseConnectionParams(
            url=f"{url}/sse",
            headers={"X-User-Id": "research_assistant"}
        )
    )

# Create the main agent
root_agent = Agent(
    name="research_assistant",
    model="gemini-2.5-flash",
    description="A comprehensive research assistant with web search, calculations, and time info",
    instruction=SYSTEM_PROMPT,
    tools=[get_mcp_toolset(), get_current_time, calculator]
)

if __name__ == "__main__":
    logger.info("Research Assistant Agent initialized successfully!")
```