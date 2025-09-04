---
title: Multi-Agent Systems
nav_order: 4
---

# Agent Guide Part 2: Multi Agent Pattern

Multi-agent systems allow you to create complex AI applications by coordinating multiple specialized agents. This guide covers everything from basic delegation to advanced orchestration patterns.

## Why Multi-Agent Systems?

Multi-agent architectures offer:
- **Specialization**: Each agent focuses on specific tasks
- **Scalability**: Add new capabilities by adding agents
- **Modularity**: Swap or update agents independently
- **Parallel Processing**: Multiple agents working simultaneously
- **Fault Tolerance**: If one agent fails, others continue

## Core Concepts

### Agent Types in ADK

1. **Task Agents**: Perform specific actions (main workers)
2. **Sequential Agents**: Execute agents in order
3. **Parallel Agents**: Run multiple agents simultaneously
4. **Loop Agents**: Iterate over data or repeat tasks
5. **Router Agents**: Dynamically choose which agent to use

## Part 1: Basic Multi-Agent Patterns

### Pattern 1: Delegation Chain

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

## Part 2: Advanced Orchestration

### Hierarchical Agent System

```python
from google.adk.agents import Agent, SequentialAgent, ParallelAgent
from typing import List, Dict, Any
import json

class HierarchicalAgentSystem:
    """Multi-level agent hierarchy for complex tasks."""
    
    def __init__(self):
        # Level 3: Worker agents (bottom level)
        self.workers = self.create_worker_agents()
        
        # Level 2: Manager agents
        self.managers = self.create_manager_agents()
        
        # Level 1: Director agent (top level)
        self.director = self.create_director_agent()
    
    def create_worker_agents(self) -> Dict[str, Agent]:
        """Create specialized worker agents."""
        return {
            "data_collector": Agent(
                name="data_collector",
                model="gemini-2.5-flash",
                description="Collects data from various sources",
                instruction="Gather comprehensive data from all available sources.",
                tools=[web_scraper, api_caller, database_query]
            ),
            "data_processor": Agent(
                name="data_processor",
                model="gemini-2.5-flash",
                description="Processes and cleans data",
                instruction="Clean, normalize, and prepare data for analysis.",
                tools=[data_cleaner, normalizer, validator]
            ),
            "analyzer": Agent(
                name="analyzer",
                model="gemini-2.5-flash",
                description="Performs data analysis",
                instruction="Analyze data and extract meaningful insights.",
                tools=[statistics_tool, ml_model, visualization]
            ),
            "report_writer": Agent(
                name="report_writer",
                model="gemini-2.5-flash",
                description="Writes reports",
                instruction="Create clear, professional reports.",
                tools=[formatter, chart_creator, pdf_generator]
            )
        }
    
    def create_manager_agents(self) -> Dict[str, Agent]:
        """Create manager agents that coordinate workers."""
        
        # Data team manager
        data_manager = SequentialAgent(
            name="data_manager",
            description="Manages data collection and processing",
            agents=[
                self.workers["data_collector"],
                self.workers["data_processor"]
            ]
        )
        
        # Analysis team manager
        analysis_manager = Agent(
            name="analysis_manager",
            model="gemini-2.5-flash",
            description="Coordinates analysis tasks",
            instruction="""You manage the analysis team. 
            Break down analysis requests into subtasks and delegate to analysts.
            Consolidate results into coherent insights.""",
            tools=[self.workers["analyzer"]]
        )
        
        # Reporting team manager
        reporting_manager = Agent(
            name="reporting_manager",
            model="gemini-2.5-flash",
            description="Manages report creation",
            instruction="Coordinate report writing and ensure quality.",
            tools=[self.workers["report_writer"]]
        )
        
        return {
            "data": data_manager,
            "analysis": analysis_manager,
            "reporting": reporting_manager
        }
    
    def create_director_agent(self) -> Agent:
        """Create top-level director agent."""
        return Agent(
            name="director",
            model="gemini-2.5-pro",  # Use more powerful model for director
            description="Executive director coordinating all teams",
            instruction="""You are the executive director of a data analysis firm.
            
            Your responsibilities:
            1. Understand client requirements
            2. Create project plan
            3. Delegate to appropriate managers
            4. Ensure quality and timeliness
            5. Deliver final results to client
            
            You have three managers:
            - Data Manager: Handles data collection and processing
            - Analysis Manager: Performs analysis and insights
            - Reporting Manager: Creates final deliverables
            
            Coordinate their work to deliver exceptional results.""",
            tools=[self.managers["data"], self.managers["analysis"], self.managers["reporting"]]
        )
    
    def execute_project(self, project_description: str) -> Dict[str, Any]:
        """Execute a complete project through the hierarchy."""
        return self.director.run(project_description)
```