---
title: Multi-Agent Systems
layout: default
nav_order: 4
---

# Multi-Agent Systems

This guide covers delegation, parallel processing, and hierarchical orchestration. These are only the most basic agent patterns. Refer to Google's ADK documentation found on the top right of the page to learn more or in this [link](https://google.github.io/adk-docs/agents/multi-agents/#c-explicit-invocation-agenttool)

## Why Multi-Agent Systems?

### Benefits

| Benefit | Description | Example |
|---------|-------------|----------|
| **Specialization** | Each agent masters specific tasks | Research agent + Writer agent |
| **Scalability** | Add capabilities without rewriting | Plugin new agents anytime |
| **Modularity** | Update agents independently | Swap models per agent |
| **Parallel Processing** | Multiple agents work simultaneously | 10x faster processing |
| **Fault Tolerance** | System continues if one fails | Resilient architecture |

## Core Concepts

### Agent Hierarchy (Parent/Sub-Agents)

The foundation for structuring multi-agent systems is the parent-child relationship defined in BaseAgent.

**Establishing Hierarchy**: You create a tree structure by passing a list of agent instances to the `sub_agents` argument when initializing a parent agent. ADK automatically sets the `parent_agent` attribute on each child agent during initialization.

**Single Parent Rule**: An agent instance can only be added as a sub-agent once. Attempting to assign a second parent will result in a ValueError.

**Importance**: This hierarchy defines the scope for Workflow Agents and influences the potential targets for LLM-Driven Delegation. You can navigate the hierarchy using `agent.parent_agent` or find descendants using `agent.find_agent(name)`.

```python
# Conceptual Example: Defining Hierarchy
from google.adk.agents import LlmAgent, BaseAgent

# Define individual agents
greeter = LlmAgent(name="Greeter", model="gemini-2.0-flash")
task_doer = BaseAgent(name="TaskExecutor") # Custom non-LLM agent

# Create parent agent and assign children via sub_agents
coordinator = LlmAgent(
    name="Coordinator",
    model="gemini-2.0-flash",
    description="I coordinate greetings and tasks.",
    sub_agents=[ # Assign sub_agents here
        greeter,
        task_doer
    ]
)

# Framework automatically sets:
# assert greeter.parent_agent == coordinator
# assert task_doer.parent_agent == coordinator
```

## Basic Multi-Agent Patterns

### Pattern 1: Delegation Chain

Use when tasks need to flow through multiple specialists.

```python
from google.adk.agents import Agent, SequentialAgent

# Specialized agents
research_agent = Agent(
    name="researcher",
    model="gemini-2.5-flash",
    description="Researches and gathers information",
    instruction="You are a research specialist. Find accurate, relevant information.",
    tools=[web_search_tool]
)

analysis_agent = Agent(
    name="analyst",
    model="gemini-2.5-flash",
    description="Analyzes data and provides insights",
    instruction="You are a data analyst. Analyze information and extract insights.",
    tools=[data_analysis_tool]
)

writer_agent = Agent(
    name="writer",
    model="gemini-2.5-flash",
    description="Creates well-written content",
    instruction="You are a professional writer. Create clear, engaging content.",
    tools=[formatting_tool]
)

# Sequential orchestration
report_generator = SequentialAgent(
    name="report_generator",
    description="Generates comprehensive reports",
    agents=[research_agent, analysis_agent, writer_agent]
)
```

### Pattern 2: Parallel Processing

Use when multiple tasks can run simultaneously.

```python
from google.adk.agents import ParallelAgent

# Create parallel agent for multi-source research
multi_source_researcher = ParallelAgent(
    name="multi_source_researcher",
    description="Research from multiple sources simultaneously",
    agents=[
        web_search_agent,
        database_query_agent,
        api_fetch_agent
    ]
)

# Usage in a larger system
comprehensive_agent = SequentialAgent(
    name="comprehensive_researcher",
    agents=[
        multi_source_researcher,  # Parallel research
        data_consolidation_agent,  # Combine results
        summary_agent  # Summarize findings
    ]
)
```
