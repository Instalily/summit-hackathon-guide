---
title: Multi-Agent Systems
layout: default
nav_order: 4
---

# Multi-Agent Systems

Learn how to build sophisticated AI systems by coordinating multiple specialized agents. This guide covers delegation, parallel processing, and hierarchical orchestration.

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

### Agent Types in ADK

| Agent Type | Purpose | Use Case |
|------------|---------|----------|
| **Task Agents** | Perform specific actions | Data processing, API calls |
| **Sequential Agents** | Execute in order | Step-by-step workflows |
| **Parallel Agents** | Run simultaneously | Multi-source data gathering |
| **Loop Agents** | Iterate over data | Batch processing |
| **Router Agents** | Dynamic selection | Intent-based routing |

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

### Pattern 3: Dynamic Routing

Use when you need to choose agents based on input.

```python
from google.adk.agents import Agent
from typing import Dict, Any

class RouterAgent:
    """Dynamic agent router based on input."""
    
    def __init__(self):
        self.agents = {
            "technical": self.create_technical_agent(),
            "creative": self.create_creative_agent(),
            "analytical": self.create_analytical_agent()
        }
        
        self.router = Agent(
            name="router",
            model="gemini-2.5-flash",
            description="Routes requests to appropriate agents",
            instruction="""Analyze the user's request and determine which type of agent to use:
            - technical: Programming, engineering, technical questions
            - creative: Writing, design, artistic tasks
            - analytical: Data analysis, math, statistics
            
            Respond with only the agent type."""
        )
    
    def create_technical_agent(self) -> Agent:
        return Agent(
            name="technical_expert",
            model="gemini-2.5-flash",
            description="Handles technical queries",
            instruction="You are a technical expert. Provide detailed, accurate technical answers.",
            tools=[code_execution_tool, documentation_search_tool]
        )
    
    def create_creative_agent(self) -> Agent:
        return Agent(
            name="creative_expert",
            model="gemini-2.5-flash",
            description="Handles creative tasks",
            instruction="You are a creative professional. Generate innovative, engaging content.",
            tools=[image_generation_tool, writing_assistant_tool]
        )
    
    def create_analytical_agent(self) -> Agent:
        return Agent(
            name="analytical_expert",
            model="gemini-2.5-flash",
            description="Handles analytical tasks",
            instruction="You are a data analyst. Provide thorough analysis and insights.",
            tools=[calculator_tool, statistics_tool, visualization_tool]
        )
    
    def route_request(self, user_input: str) -> Dict[str, Any]:
        """Route request to appropriate agent."""
        # Determine which agent to use
        agent_type = self.router.run(user_input)
        
        # Get the appropriate agent
        selected_agent = self.agents.get(agent_type, self.agents["technical"])
        
        # Execute with selected agent
        result = selected_agent.run(user_input)
        
        return {
            "agent_used": agent_type,
            "result": result
        }
```

## Real-World Examples

### Example 1: Customer Support System

```python
class CustomerSupportSystem:
    """Multi-agent customer support with specialized handlers."""
    
    def __init__(self):
        # Specialized support agents
        self.billing_agent = Agent(
            name="billing_specialist",
            model="gemini-2.5-flash",
            instruction="Handle billing inquiries, refunds, and payment issues."
        )
        
        self.technical_agent = Agent(
            name="technical_support",
            model="gemini-2.5-flash",
            instruction="Resolve technical issues and provide troubleshooting."
        )
        
        self.general_agent = Agent(
            name="general_support",
            model="gemini-2.5-flash",
            instruction="Handle general inquiries and FAQs."
        )
        
        # Router to classify intent
        self.intent_router = Agent(
            name="intent_classifier",
            model="gemini-2.5-flash",
            instruction="""Classify customer inquiries into:
            - billing: Payment, refunds, invoices
            - technical: Bugs, errors, not working
            - general: Other questions
            Return only the category."""
        )
    
    def handle_inquiry(self, customer_message: str):
        # Route to appropriate agent
        intent = self.intent_router.run(customer_message)
        
        if "billing" in intent.lower():
            return self.billing_agent.run(customer_message)
        elif "technical" in intent.lower():
            return self.technical_agent.run(customer_message)
        else:
            return self.general_agent.run(customer_message)
```

### Example 2: Content Creation Pipeline

```python
class ContentCreationPipeline:
    """Automated content creation with multiple specialists."""
    
    def __init__(self):
        self.pipeline = SequentialAgent(
            name="content_pipeline",
            agents=[
                Agent(name="researcher", instruction="Research the topic thoroughly"),
                Agent(name="outliner", instruction="Create detailed content outline"),
                Agent(name="writer", instruction="Write engaging content"),
                Agent(name="editor", instruction="Edit for clarity and grammar"),
                Agent(name="seo_optimizer", instruction="Optimize for search engines")
            ]
        )
    
    def create_article(self, topic: str):
        return self.pipeline.run(f"Create article about: {topic}")
```

## Best Practices

### Do's

1. **Keep agents focused** - One clear responsibility per agent
2. **Use appropriate models** - Heavy tasks → Pro models, Simple tasks → Flash models
3. **Implement error handling** - Graceful fallbacks for agent failures
4. **Monitor performance** - Track latency and costs
5. **Test incrementally** - Start simple, add complexity gradually

### Don'ts

1. **Over-engineer** - Don't use multi-agent for simple tasks
2. **Ignore costs** - Multiple agents = multiple API calls
3. **Skip validation** - Always validate inter-agent data
4. **Tight coupling** - Keep agents loosely coupled
5. **Neglect logging** - Debug logs are crucial

## Performance Optimization

### Parallel vs Sequential Processing

```python
# Slow: Sequential execution (3+ seconds)
results = []
for agent in agents:
    results.append(agent.run(task))

# Fast: Parallel execution (1 second)
parallel_agent = ParallelAgent(agents=agents)
results = parallel_agent.run(task)
```

### Model Selection Strategy

| Task Complexity | Recommended Model | Cost | Speed |
|----------------|-------------------|------|-------|
| Simple routing | gemini-2.5-flash | Low | Fast |
| Data processing | gemini-2.5-flash | Low | Fast |
| Complex reasoning | gemini-2.5-pro | High | Medium |
| Creative tasks | gemini-2.5-pro | High | Medium |

## Advanced Patterns

### Pattern 4: Feedback Loop System

```python
class FeedbackLoopSystem:
    """Agents that learn from each other's outputs."""
    
    def __init__(self):
        self.generator = Agent(name="generator", instruction="Generate content")
        self.critic = Agent(name="critic", instruction="Critique and suggest improvements")
        self.refiner = Agent(name="refiner", instruction="Refine based on feedback")
    
    def create_with_feedback(self, prompt: str, iterations: int = 2):
        result = self.generator.run(prompt)
        
        for _ in range(iterations):
            feedback = self.critic.run(f"Critique this: {result}")
            result = self.refiner.run(f"Improve based on: {feedback}\nOriginal: {result}")
        
        return result
```

### Pattern 5: Consensus System

```python
class ConsensusSystem:
    """Multiple agents vote on the best solution."""
    
    def __init__(self):
        self.experts = [
            Agent(name=f"expert_{i}", model="gemini-2.5-flash")
            for i in range(3)
        ]
        self.arbitrator = Agent(
            name="arbitrator",
            instruction="Choose the best solution from multiple options"
        )
    
    def get_consensus(self, question: str):
        # Get solutions from all experts
        solutions = ParallelAgent(agents=self.experts).run(question)
        
        # Arbitrator picks the best
        return self.arbitrator.run(f"Choose best solution:\n{solutions}")
```

## Hackathon Project Ideas

### Beginner Multi-Agent Projects

1. **Blog Generator**: Research → Outline → Write → Edit
2. **Study Assistant**: Question analyzer → Resource finder → Explainer
3. **Recipe Creator**: Ingredient checker → Recipe generator → Nutrition calculator

### Advanced Multi-Agent Projects

1. **AI Startup Assistant**: Market researcher → Business planner → Pitch creator
2. **Code Review System**: Style checker → Bug finder → Performance analyzer → Security auditor
3. **Investment Advisor**: Data collector → Risk analyzer → Portfolio optimizer → Report generator

## Summary

You've learned:
- How to design multi-agent architectures
- Parent/sub-agent hierarchy in ADK
- Sequential, parallel, and hierarchical patterns
- Dynamic routing and orchestration
- Real-world implementation examples
- Performance optimization techniques

## Resources

- [ADK Documentation](https://google.github.io/adk-docs/)
- [Example Code Repository](https://github.com/google/adk-examples)
- [Multi-Agent Design Patterns](https://patterns.adk.dev)

---

<div style="text-align: center; margin-top: 3rem;">
  <a href="getting-started" class="btn btn-purple fs-5">Back to Getting Started</a>
  <a href="flowise-quickstart" class="btn btn-primary fs-5" style="margin-left: 1rem;">Try Flowise</a>
</div>