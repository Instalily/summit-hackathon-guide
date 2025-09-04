---
title: Flowise Quickstart
layout: default
nav_order: 5
---

# Flowise Quickstart

Build AI agents using Flowise's drag-and-drop interface - no coding required.

## What is Flowise?

Flowise is a visual AI workflow builder that lets you:
- Drag and drop agent creation
- Connect multiple AI services
- Deploy instantly
- Monitor performance in real-time

## Quick Access

### Hackathon Credentials

| Service | Email | Password |
|---------|-------|----------|
| **Flowise Platform** | `innovationsummithackathon@gmail.com` | `HE!Xt13Gf4&0` |
| **Nuclia Cloud** | `innovationsummithackathon@gmail.com` | `HE!Xt13Gf4&0` |

### API Keys

| Service | Key (Truncated) | Status |
|---------|-----------------|--------|
| **OpenAI** | `sk-proj-Dd1HRW...` | Ready |
| **Perplexity** | `pplx-5yLwhLX...` | Ready |
| **Nuclia** | *Configure in Step 3* | Pending |

---

## Prerequisites

- Web browser (Chrome/Firefox recommended)
- Hackathon credentials (provided above)
- 20 minutes to complete setup

---

## Step 1: Access Flowise

### 1.1 Navigate to Platform

```
URL: https://[your-instance].flowiseai.com
```

**Note**: Your hackathon coordinator will provide the exact URL.

### 1.2 Login

1. Click **Login** button (top-right)
2. Enter credentials from table above
3. You're logged in

### Dashboard Overview

- **Dashboard**: Your main workspace
- **Marketplace**: Pre-built flow templates
- **Credentials**: API key management
- **Flows**: Your created workflows

---

## Step 2: Select Your Workspace

### 2.1 Organization Setup

```
Organization → SRS Distribution → Select
```

### 2.2 Team Workspace

```
Workspace → [Your Team Name] → Select
```

**Important**: Ensure you're in the correct team workspace before creating flows.

---

## Step 3: Configure Integrations

### 3.1 Add Nuclia Knowledge Base

1. Open Nuclia in new tab: `https://nuclia.cloud`
2. Login with hackathon credentials
3. Find Your KB:
   - Navigate to Knowledge Bases
   - Copy API Endpoint: `https://[region].nuclia.cloud/api/v1/kb/[kb-id]`
   - Copy API Key from settings

### 3.2 Configure in Flowise

1. Go to **Credentials** → **Add Credentials**
2. Add each service:

```yaml
OpenAI:
  Name: OPENAI_API_KEY
  Value: [Provided API Key]
  
Perplexity:
  Name: PERPLEXITY_API_KEY  
  Value: [Provided API Key]
  
Nuclia:
  Name: NUCLIA_API_KEY
  Value: [Your KB API Key]
  Endpoint: [Your KB Endpoint]
```

---

## Step 4: Build Your First Flow

### 4.1 Create New Flow

1. Click **New Flow**
2. Name it: "My First Agent"
3. Visual canvas will open

### 4.2 Essential Nodes

Drag these nodes onto your canvas:

| Node Type | Purpose | Configuration |
|-----------|---------|---------------|
| **Input** | Receives user queries | Set input type: Text |
| **LLM Chain** | AI processing | Model: GPT-4, Temp: 0.7 |
| **Nuclia Search** | Knowledge retrieval | Use configured credentials |
| **Output** | Returns responses | Format: JSON/Text |

### 4.3 Connect the Nodes

```
Input --> Nuclia Search --> LLM Chain --> Output
```

**How to Connect:**
1. Click output port of first node
2. Drag to input port of next node
3. Release to create connection

### 4.4 Configure LLM Chain

```javascript
// Example System Prompt
You are a helpful AI assistant with access to a knowledge base.
Use the search results to provide accurate, relevant answers.
If you don't know something, say so clearly.
Always cite your sources when using knowledge base information.
```

---

## Step 5: Test Your Flow

### 5.1 Run Test

1. Click **Test** button
2. Enter sample query: `"What is ADK?"`
3. Watch the execution flow
4. Check output response

### 5.2 Debug Tips

| Issue | Solution |
|-------|----------|
| **No response** | Check node connections |
| **API errors** | Verify credentials |
| **Slow execution** | Optimize prompt length |
| **Wrong answers** | Adjust system prompt |

---

## Flow Templates

### Available Templates

**Customer Support Bot**
- Intent classification
- FAQ retrieval
- Escalation logic
- Response formatting

**Research Assistant**
- Multi-source search
- Citation tracking
- Summary generation
- Fact verification

**Content Generator**
- Topic research
- Outline creation
- Content writing
- SEO optimization

**Data Analyzer**
- Data ingestion
- Pattern recognition
- Insight generation
- Report creation

---

## Advanced Features

### Conditional Logic

```yaml
If-Then Node:
  Condition: user_intent == "complaint"
  Then: Route to escalation flow
  Else: Continue normal flow
```

### Parallel Processing

```yaml
Parallel Node:
  Branch 1: Search documentation
  Branch 2: Query knowledge base
  Branch 3: Check FAQ database
  Merge: Combine all results
```

### Error Handling

```yaml
Try-Catch Node:
  Try: Execute main flow
  Catch: Log error, return fallback
  Finally: Clean up resources
```

---

## Best Practices

### Do's

1. **Start Simple**: Basic input → process → output
2. **Test Often**: Validate each node before adding more
3. **Use Templates**: Learn from marketplace examples
4. **Document Flows**: Add descriptions to nodes
5. **Save Regularly**: Use Ctrl+S frequently

### Don'ts

1. **Over-complicate**: Keep flows focused
2. **Skip Testing**: Always test before deployment
3. **Ignore Errors**: Handle all edge cases
4. **Hard-code Values**: Use environment variables
5. **Forget Backups**: Export flows regularly

---

## Troubleshooting

### Common Issues

**Connection Failed**
- Solution: Check API keys and network connection

**Slow Performance**
- Solution: Reduce prompt size, optimize node count

**Unexpected Output**
- Solution: Review system prompts and node logic

---

## Performance Optimization

### Speed Tips

| Optimization | Impact | Implementation |
|--------------|--------|----------------|
| **Cache Results** | 50% faster | Enable caching in nodes |
| **Batch Processing** | 3x throughput | Use batch input nodes |
| **Async Execution** | 2x speed | Enable parallel processing |
| **Minimize Hops** | 30% faster | Reduce node count |

---

## Hackathon Project Ideas

### Using Flowise

1. **Smart FAQ Bot**: Build a visual FAQ system with intelligent routing
2. **Multi-Model Orchestrator**: Combine GPT-4, Perplexity, and Nuclia
3. **Workflow Automator**: Create complex business process flows
4. **Knowledge Navigator**: Visual knowledge base explorer
5. **Decision Tree Builder**: Interactive decision support system

---

## Resources

### Documentation
- [Flowise Docs](https://docs.flowiseai.com)
- [Nuclia API Reference](https://docs.nuclia.dev)
- [OpenAI Platform](https://platform.openai.com/docs)

### Getting Help
- Ask mentors in Discord
- Check #flowise-help channel
- Email support team
- Search documentation

---

## Next Steps

### Completed
- Logged into Flowise
- Configured integrations
- Built first flow
- Tested successfully

### Now Try
1. Explore marketplace templates
2. Combine multiple AI models
3. Add error handling
4. Implement conditional logic
5. Deploy your flow

---

<div style="text-align: center; margin-top: 3rem;">
  <a href="getting-started" class="btn btn-purple fs-5">Back to ADK Guide</a>
  <a href="multi-agent-systems" class="btn btn-primary fs-5" style="margin-left: 1rem;">Learn Multi-Agent Systems</a>
</div>