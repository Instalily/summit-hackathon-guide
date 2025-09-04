---
title: First Agent
nav_order: 3
---

# Agent Guide Part 1: Single Agent Pattern

In this tutorial, we'll build a complete AI agent from scratch using Google's Agent Development Kit (ADK). By the end, you'll have a working agent that can answer questions, use tools, and interact with users.

## What We'll Build

We'll create a "Research Assistant" agent that can:
- Answer questions using an LLM
- Search the web for information
- Provide the current time
- Cite sources for its answers

## Prerequisites

- Completed the [Getting Started Guide](../getting-started.md)
- Python environment set up with ADK installed
- Basic understanding of Python functions

## Part 1: Understanding Agent Components

An ADK agent consists of four main components:

### 1. Agent Definition
The core configuration that defines your agent's identity and capabilities.

### 2. Model Selection
The LLM that powers your agent's reasoning (Gemini, GPT, Claude, etc.).

### 3. Instructions/Prompts
The system prompt that guides your agent's behavior and personality.

### 4. Tools
Functions your agent can call to perform actions or retrieve information.

## Part 2: Creating a Basic Agent

Let's start with the simplest possible agent.

### Step 1: Create the Agent File

Create a new file `my_first_agent.py`:

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

### Step 2: Run Your Agent

```bash
# Save the file and run
adk web
```

Cmd click on the link and start chatting with your agent!

Example prompts to try:
- "Hello! Who are you?"
- "What can you help me with?"
- "Explain quantum computing in simple terms"

## Part 3: Adding Custom Tools

Tools give your agent superpowers. Let's add some capabilities.

### Step 1: Create a Time Tool

Update `my_first_agent.py`:

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

Restart your agent and try:
- "What time is it in Tokyo?"
- "Tell me the current time in London and New York"
- "What's the date in Sydney?"

## Part 4: Adding a Calculator Tool

Let's add another useful tool:

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

## Part 5: Enhancing with Better Prompts

Good prompts make agents more effective. Let's create a separate prompt file.

### Step 1: Create `prompts.py`

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

## Part 8: Testing Your Complete Agent

Try these prompts to test all capabilities:

### Basic Conversation
- "Hello! What can you help me with?"
- "Explain how you work"

### Time Queries
- "What time is it in Tokyo and London?"
- "What day is it in Sydney?"

### Calculations
- "Calculate 15% tip on $85.50"
- "What's the square root of 2024?"
- "Convert 100 fahrenheit to celsius using the formula (F-32)*5/9"

### Web Search (via MCP)
- "What's the latest news in AI?"
- "Tell me about recent developments in quantum computing"
- "Search for Python programming best practices"

### Combined Tasks
- "What time is it in New York, and what's 20% of 150?"
- "Search for the population of Tokyo and calculate its square root"

## Part 9: Debugging and Troubleshooting

### Enable Detailed Logging

```python
import logging

# At the top of your file
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# In your tools
logger.debug(f"Received request for city: {city}")
logger.info(f"Returning time: {now}")
```

### Common Issues

**Tool not being called:**
- Check your instruction prompt mentions the tool
- Verify tool is in the tools list
- Make sure tool has proper documentation

**MCP connection failing:**
- Check internet connection
- Verify the server URL
- Look for error messages in logs

**Agent not responding as expected:**
- Review your system prompt
- Check if it's too restrictive or contradictory
- Test with simpler prompts first

## Part 10: Next Steps

### Enhancements to Try

1. **Add More Cities** to the timezone tool
2. **Create a Weather Tool** using an API
3. **Add Memory** to remember user preferences
4. **Implement Data Visualization** tools
5. **Create Specialized Prompts** for different use cases

### Advanced Topics to Explore

- [Working with Tools](./02-working-with-tools.md) - Deep dive into tool creation
- [MCP Integration](./03-mcp-integration.md) - Advanced MCP features
- [Multi-Agent Systems](../guides/multi-agent-systems.md) - Coordinating multiple agents

## Summary

Congratulations! You've built a complete ADK agent with:
- ✅ Custom tools for time and calculations
- ✅ MCP integration for web search
- ✅ Professional system prompts
- ✅ Error handling and logging
- ✅ Interactive web interface

This agent can serve as a foundation for more complex projects. The patterns you've learned here apply to any ADK agent you build.

## Challenge Exercises

1. **Add a Wikipedia Tool**: Create a tool that fetches Wikipedia summaries
2. **Implement Unit Conversion**: Extend the calculator to convert units
3. **Create a Note-Taking Tool**: Allow the agent to save and retrieve notes
4. **Add Language Translation**: Integrate a translation API
5. **Build a Stock Price Tool**: Fetch real-time stock information

Keep experimenting and building! The ADK framework is designed to grow with your needs.

---

Ready for more? Continue to [Working with Tools](./02-working-with-tools.md) →