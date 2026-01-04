# Amazon Bedrock Complete Study Guide
## AWS Certified AI Practitioner Exam Preparation

---

# Table of Contents
1. [Inference & APIs](#1-inference--apis)
2. [Knowledge Bases (RAG)](#2-knowledge-bases-rag)
3. [Agents](#3-agents)
4. [Guardrails](#4-guardrails)
5. [Model Customization](#5-model-customization)
6. [Flows & Prompt Management](#6-flows--prompt-management)

---

# 1. Inference & APIs

## Core API Comparison

### InvokeModel vs Converse API

| Aspect | InvokeModel | Converse API |
|--------|-------------|--------------|
| **Purpose** | Direct, low-level model invocation | Unified conversational interface |
| **Format** | Model-specific request/response | Model-agnostic format |
| **Best For** | Embeddings, images, model-specific features | Chatbots, multi-turn, tool use |
| **Streaming** | InvokeModelWithResponseStream | ConverseStream |

**Key Insight**: Converse API automatically applies conversation templates for Mistral AI and Meta models.

**Limitation**: Converse API does NOT support embedding or image generation models.

### API Selection Guide

| Use Case | Recommended API |
|----------|-----------------|
| Multi-turn conversations | Converse |
| Tool use / function calling | Converse |
| Model-agnostic code | Converse |
| Embeddings generation | InvokeModel |
| Image generation | InvokeModel |

---

## Inference Parameters

### Randomness Controls
- **Temperature**: Lower = deterministic, Higher = creative
- **Top K**: Limits to K most probable tokens
- **Top P (Nucleus)**: Limits to tokens comprising top P% probability mass

**Note**: Default values vary significantly by model - always check model docs.

---

## Tool Use (Function Calling)

**Critical**: The model ONLY requests tool calls - it NEVER executes them. Your application handles all execution.

```
User Query → Model → Tool Request → Your Code Executes → Tool Result → Model → Final Response
```

---

## Prompt Caching

### Mechanics
- **TTL**: 5 minutes (resets on each hit)
- **Minimum tokens**: 1,024 - 4,096 (model-dependent)
- **Maximum checkpoints**: 4 per request

### Model Requirements

| Model | Min Tokens | Cacheable Fields |
|-------|------------|------------------|
| Claude Opus 4.5 | 4,096 | system, messages, tools |
| Claude 3.7 Sonnet | 1,024 | system, messages, tools |
| Amazon Nova | 1,000 | system, messages (text only) |

**Best Practice**: Place cache checkpoints at END of static content.

---

## Batch Inference

- Use `CreateModelInvocationJob` for large-scale processing
- **Not supported** with provisioned throughput
- More cost-efficient than individual API calls

---

## Inference Profiles

### Types

| Type | Description |
|------|-------------|
| **System-defined (Cross-Region)** | Pre-configured multi-region routing |
| **Application (Single-Region)** | User-created for tracking |
| **Application (Multi-Region)** | User-created using system profile |

**Pricing**: Based on calling region, NOT destination region.

---

## Cross-Region Inference

| Mode | Data Boundaries | Use Case |
|------|-----------------|----------|
| **Geographic** | US, EU, APAC only | Compliance/data residency |
| **Global** | Any region | Max throughput, ~10% savings |

**Security**: No public internet traversal - stays on AWS internal network.

**Important**: Can route to regions NOT manually enabled in your account.

---

## Service Tiers

| Tier | Performance | Cost | Use Case |
|------|-------------|------|----------|
| **Reserved** | Guaranteed | Fixed premium | Mission-critical |
| **Priority** | Fastest on-demand | Higher | Customer-facing |
| **Standard** | Baseline | Standard | General tasks |
| **Flex** | May queue | Lowest | Batch processing |

**Key**: On-demand quota is SHARED across priority, standard, and flex tiers.

---

## Latency-Optimized Inference (Preview)

**Supported Models** (limited):
- Amazon Nova Pro (us-east-1, us-east-2)
- Claude 3.5 Haiku (us-east-2, us-west-2)
- Llama 3.1 405B/70B (us-east-2, us-west-2)

**Tradeoff**: Falls back to standard inference when quota reached.

---

# 2. Knowledge Bases (RAG)

## How RAG Works in Bedrock

### Two Phases

**Phase 1: Pre-processing (Ingestion)**
```
Raw Data → Parsing → Chunking → Embedding → Vector Indexing
```

**Phase 2: Runtime (Query)**
```
User Query → Query Embedding → Semantic Search → Context Augmentation → LLM Response
```

### API Operations

| Operation | Purpose |
|-----------|---------|
| **Retrieve** | Returns raw source chunks (custom pipelines) |
| **RetrieveAndGenerate** | Full RAG with citations |
| **GenerateQuery** | Natural language to SQL |

---

## Vector Store Options

| Store | Key Characteristics |
|-------|---------------------|
| **OpenSearch Serverless** | Default, zero setup, serverless |
| **Pinecone** | High performance, purpose-built |
| **Neptune** | Graph-based, relationship-aware |
| **Aurora RDS** | SQL compatibility |
| **Redis Enterprise** | Low latency, real-time |
| **MongoDB Atlas** | Flexible schema |

---

## Chunking Strategies

### Comparison

| Strategy | Best For | Trade-off |
|----------|----------|-----------|
| **Default (~300 tokens)** | Quick setup | May not optimize for content |
| **Fixed-Size** | Uniform content | Configurable overlap |
| **Hierarchical** | Large documents | Fewer results, more context |
| **Semantic** | Complex relationships | Additional FM costs |
| **No Chunking** | Pre-processed files | No page-level citations |

### Hierarchical Chunking Details
- Parent chunks (large context) + Child chunks (precise embeddings)
- Child retrieved first, then replaced with parent
- **Not recommended** with S3 vector bucket

### Semantic Chunking Details
- Divides by meaningful boundaries
- Buffer size of 1 = 3 sentences (previous, current, next)
- Larger buffers capture more context but may introduce noise

---

## Data Source Connectors

| Connector | Multimodal Support |
|-----------|-------------------|
| **Amazon S3** | Yes |
| **Custom Data Source** | Yes |
| **Confluence** | No |
| **SharePoint** | No |
| **Salesforce** | No |
| **Web Crawler** | No |

**Critical**: Multimodal content ONLY works with S3 and Custom connectors.

---

## Embedding Models

| Model | Dimensions | Vector Types |
|-------|------------|--------------|
| Titan Embeddings G1 - Text | 1536 | float32 |
| Titan Text Embeddings V2 | 256/512/1024 | float32, binary |
| Cohere Embed | 1024 | float32, binary |

**Binary vectors**: Lower storage cost, reduced precision.

---

## Advanced Parsing

| Parser | Cost | Multimodal | Best For |
|--------|------|------------|----------|
| **Default** | Free | No | Text extraction |
| **Bedrock Data Automation** | Per-page | Yes | PDF/images |
| **Foundation Models** | Per-token | Yes | Custom extraction |

**Warning**: Once you select advanced parsing, it applies to ALL PDFs in data source.

---

## Multimodal Capabilities

### Processing Approaches

| Approach | Converts To | Best For |
|----------|-------------|----------|
| **Nova Multimodal Embeddings** | Native vectors | Visual/audio similarity |
| **Bedrock Data Automation** | Text | Text-based search |

**Timestamp Handling**: Your application must extract segments based on start/end metadata.

---

## Reranking

**Purpose**: Reorder retrieved results by relevance.

**Benefits**:
- Fewer, more relevant results
- Decreased cost and latency
- More accurate responses

**Limitation**: Text only - no images, audio, or video.

---

# 3. Agents

## Architecture Overview

### Core Components
- **Foundation Models**: Decision engine
- **Action Groups**: Executable capabilities
- **Knowledge Bases**: Contextual augmentation

### Runtime Process Flow
```
User Input → Pre-processing (optional) → Orchestration Loop → Post-processing (optional) → Response
```

**Orchestration Loop**:
1. FM generates rationale
2. FM predicts action group OR KB query
3. Execute action (Lambda or return control)
4. Generate observation
5. Re-augment prompt
6. Loop until complete

---

## Action Groups

### Two Definition Methods

| Method | Best For |
|--------|----------|
| **OpenAPI Schema** | Explicit API mapping |
| **Function Details** | Simplified setup |

### Lambda Integration
- Max 11 API operations per action group
- ONE Lambda function per group
- Payload limited to 6 MB

### Return Control Pattern
When you need control over execution:
1. Configure `RETURN_CONTROL`
2. Agent returns parameters with `invocationId`
3. Your app executes action
4. Send results back with same `invocationId`

**Note**: If `returnControlInvocationResults` included, `inputText` is ignored.

---

## Multi-Agent Collaboration

### Architecture
```
Supervisor Agent (orchestrator)
    ├── Collaborator Agent 1
    ├── Collaborator Agent 2
    └── Collaborator Agent N
```

**Key**: Hierarchical model with supervisor routing to domain specialists.

---

## Memory System

| Type | Persistence | Scope |
|------|-------------|-------|
| **Session Context** | Session duration | Same `sessionId` |
| **Memory Context** | 1-365 days | Same `memoryId` |

**Association Trigger**: Memory associates when `endSession=true` OR `idleSessionTimeout` expires.

---

## Code Interpretation

**Capabilities**: Generate, run, troubleshoot code in sandbox.

**Limits**:
- Max 5 files per request
- Total 10 MB file size
- **25 concurrent sessions per account**

**Regions**: us-east-1, us-west-2, eu-central-1

---

## Orchestration Strategies

| Strategy | Use Case |
|----------|----------|
| **Default (ReAct)** | Standard workflows |
| **Advanced Prompts** | Modified templates |
| **Custom Orchestration** | Lambda-based complex logic |

### Four Base Prompt Templates
1. Pre-processing (disabled by default)
2. Orchestration
3. KB response generation
4. Post-processing (disabled by default)

**Critical Warning**: Agent instructions IGNORED if: single KB + default prompts + no action groups + user input disabled.

---

## Session State

| Type | Persistence |
|------|-------------|
| **sessionAttributes** | Full session |
| **promptSessionAttributes** | Single turn |
| **conversationHistory** | Multi-agent flows |

---

# 4. Guardrails

## How Guardrails Work

### Processing Pipeline
```
User Input → Parallel Policy Evaluation → BLOCKED (no inference charges) OR PASSES → Model → Evaluation → Response
```

**Key**: If input blocked, model inference skipped = no charges.

---

## Safeguard Types

| Type | Purpose | Best For |
|------|---------|----------|
| **Content Filters** | Block harmful categories | General safety |
| **Denied Topics** | Block specific topics | Domain restrictions |
| **Contextual Grounding** | Detect hallucinations | RAG applications |
| **Automated Reasoning** | Validate logical rules | Regulated industries |
| **Sensitive Info Filters** | Block/mask PII | Privacy compliance |
| **Word Filters** | Block exact words | Profanity, terms |

---

## Content Filter Categories
- Hate
- Insults
- Sexual
- Violence
- Misconduct
- Prompt Attack

### Filter Strength Configuration
- Start with **Medium**, adjust based on false positive rate
- Different thresholds for inputs vs outputs
- Actions: `BLOCK` or `NONE` (detect-only)

---

## Denied Topics

**Limits**:
- Up to 30 topics per guardrail
- 200 chars (Classic) or 1,000 chars (Standard) per definition
- Up to 5 sample phrases per topic

**Best Practices**:
- Define topics crisply and precisely
- Don't include examples in definitions
- Don't define negative topics
- Use word filters for individual words

---

## Contextual Grounding Check

**Two Checks**:
1. **Grounding**: Response accurate based on source?
2. **Relevance**: Response answers the query?

**Limits**:
- Source: 100,000 chars max
- Query: 1,000 chars max
- Response: 5,000 chars max
- Threshold: 0 to 0.99 (1.0 invalid)

**Not suitable for**: Conversational chatbots.

---

## Automated Reasoning

**Purpose**: Validate against logical rules (HR policies, loan approvals, regulations).

**Limitations**:
- English (US) only
- No streaming API support
- Limited regional availability
- Cannot detect prompt injection

**Variable Types**: BOOL, INT, NUMBER, enum

---

## Tier Comparison

| Feature | Standard | Classic |
|---------|----------|---------|
| Multi-language | Extensive | English, French, Spanish |
| Cross-Region | Supported | Not supported |
| Prompt Leakage Detection | Yes | No |
| Denied Topic Definition | 1,000 chars | 200 chars |

**Recommendation**: Use Standard tier for new deployments.

---

## Input Tagging (Required for Prompt Attack Detection)

```xml
<amazon-bedrock-guardrails-guardContent_xyz>
  User input goes here
</amazon-bedrock-guardrails-guardContent_xyz>
```

---

# 5. Model Customization

## Customization Methods Overview

| Method | Data Type | Purpose |
|--------|-----------|---------|
| **Distillation** | Prompts (unlabeled) | Transfer from teacher to student |
| **Supervised Fine-Tuning** | Labeled pairs | Task-specific accuracy |
| **Continued Pre-Training** | Unlabeled text | Domain knowledge expansion |
| **Reinforcement Fine-Tuning** | Prompts + reward function | Optimize for outcomes |

---

## Model Distillation

**How It Works**:
1. Provide input prompts (or use invocation logs)
2. Bedrock generates responses from teacher model
3. Student model fine-tuned on synthetic data

**Two Paths**:
- **Path A**: Upload prompts, Bedrock generates teacher responses
- **Path B**: Use existing CloudWatch Logs from production

**Key**: Can expand dataset up to 15,000 prompt-response pairs.

---

## Supervised Fine-Tuning

**Data Formats**:

**Non-conversational**:
```json
{"prompt": "Question?", "completion": "Answer."}
```

**Conversational**:
```json
{
  "schemaVersion": "bedrock-conversation-2024",
  "system": [{"text": "You are helpful"}],
  "messages": [
    {"role": "user", "content": [{"text": "Q"}]},
    {"role": "assistant", "content": [{"text": "A"}]}
  ]
}
```

---

## Continued Pre-Training

**Data Format**:
```json
{"input": "Your domain text here..."}
```

**Use Cases**: Private documents, domain vocabulary, proprietary knowledge.

**Supported Models**: Titan Text G1 - Express only.

---

## Reinforcement Fine-Tuning (RFT)

**Three Stages**:
1. Response Generation (4 responses per prompt)
2. Reward Computation (Lambda scoring)
3. Actor Training (GRPO optimization)

**Two Approaches**:

| Approach | Best For |
|----------|----------|
| **RLVR** (Verifiable Rewards) | Code, math, objective tasks |
| **RLAIF** (AI Feedback) | Subjective quality |

---

## Training Data Requirements

| Model | Min Records | Max Records | Max Tokens |
|-------|-------------|-------------|------------|
| Claude 3 Haiku | - | 10,000 | 4,096 input + 2,048 output |
| Llama 3.1/3.2 | 5 | 10,000 | 16,000 total |
| Titan Text | 1,000 | 500,000 | 4,096 |

**Token Estimation**: Use 6 characters per token.

**File Size Limits**:
- Training: 1 GB (fine-tuning), 10 GB (continued pre-training)
- Validation: 100 MB

---

## Hyperparameters (Key Differences)

| Model | Epoch Default | Batch Size | Special Features |
|-------|---------------|------------|------------------|
| Claude 3 Haiku | 2 | 32 (4-256) | Early stopping |
| Llama 3.1 | 5 | 1 (fixed) | Wide learning rate range |
| Nova | 2 | - | Warmup steps |

**Key Details**:
- Claude has automatic early stopping (overfitting prevention)
- Llama batch size is LOCKED at 1
- Each epoch multiplies token processing cost

---

## Provisioned Throughput

**MANDATORY for all custom models**.

| Commitment | Discount |
|------------|----------|
| No commitment | None |
| 1 month | Moderate |
| 6 months | Maximum |

---

## Regional Constraints

| Model Family | Distillation Region |
|--------------|---------------------|
| Claude & Llama | US West (Oregon) |
| Nova | US East (N. Virginia) |

**Note**: Nova models cannot be copied to other regions.

---

# 6. Flows & Prompt Management

## Bedrock Flows

### Architecture
Directed node-based workflows: Input → Processing Nodes → Output

**Key Elements**:
- **Nodes**: Processing units
- **Connections**: Data (solid) or Conditional (dotted)
- **Expressions**: JSONPath-based data extraction (`$.data.field`)

---

## Node Types

| Category | Node | Purpose |
|----------|------|---------|
| **Logic** | Input | Entry point (exactly one) |
| | Output | Return response (multiple allowed) |
| | Condition | Route by logic (first match wins) |
| | Iterator | Process arrays (sequential) |
| | DoWhile | Loop (max 10 iterations) |
| **AI** | Prompt | Model inference |
| | Agent | Agent orchestration (supports multi-turn) |
| | Knowledge Base | RAG queries |
| **Data** | S3 Storage/Retrieval | Read/write S3 |
| | Lambda | Custom code |
| | Inline Code | Python in-flow (max 5 nodes) |

**Key Details**:
- Iterator is SEQUENTIAL, not parallel
- Inline Code NOT supported in async execution
- S3 Retrieval requires UTF-8 encoding

---

## Multi-Turn Invocation (Preview)

Agent nodes can pause for user input:
1. First call: No `executionId`
2. Agent pauses: `INPUT_REQUIRED` event
3. Resume: Call with `executionId` + response
4. Complete: `SUCCESS` event

---

## When to Use Flows vs Agents vs Direct API

| Approach | Best For |
|----------|----------|
| **Direct API** | Simple calls, max control |
| **Agents** | Autonomous reasoning, tool use |
| **Flows** | Deterministic pipelines, visual dev |

**Hybrid**: Embed Agent nodes within Flows.

---

## Prompt Management

### Core Features
- **Variables**: `{{variable_name}}` placeholders
- **Variants**: Alternative configurations
- **Versioning**: Immutable snapshots

### Workflow
```
Create → Add Variables → Configure Model → Create Variants → Test → Save Version → Deploy
```

---

## Prompt Optimization

Bedrock can automatically rewrite prompts for better results.

**Best Practice**: Optimize in English only.

**Supported Models**: Claude 3.x, Nova, Llama 3.x, Mistral Large

---

## Intelligent Prompt Routing

Dynamically routes requests between models in same family:
1. Analyzes prompt
2. Predicts quality for each model
3. Routes to best quality/cost combo

**Limitations**:
- English only
- Same model family only
- No application-specific learning

---

## Prompt Engineering Best Practices

### Structure
```
[Context/Background]
[Input Content]
[Task/Instruction]
[Output Format]
```

### Techniques

| Technique | When to Use |
|-----------|-------------|
| **Zero-Shot** | Simple tasks |
| **Few-Shot** | Complex output, calibration needed |
| **Chain-of-Thought** | Reasoning tasks |

### Model-Specific Formatting

| Model | Format |
|-------|--------|
| Claude | `<example></example>` tags |
| Titan | `User: {}\nBot:` |
| Llama | Meta's prompting guide |

---

# Quick Reference Cards

## API Decision Tree
```
Need inference?
├── Single call, simple → InvokeModel / Converse
├── Batch processing → CreateModelInvocationJob
├── Streaming → InvokeModelWithResponseStream / ConverseStream
└── Tool use → Converse API
```

## Cost Optimization Strategies
1. **Global cross-region**: ~10% savings
2. **Flex tier**: Non-time-sensitive workloads
3. **Prompt caching**: Repeated long contexts
4. **Batch inference**: Large-scale processing
5. **Binary vectors**: Lower storage costs

## Feature Compatibility Matrix

| Feature | InvokeModel | Converse | Notes |
|---------|-------------|----------|-------|
| Text Generation | Yes | Yes | |
| Embeddings | Yes | No | |
| Image Generation | Yes | No | |
| Tool Use | No | Yes | |
| Multi-turn | Manual | Built-in | |
| Guardrails | Yes | Yes | |

## Common Exam Scenarios

| Scenario | Solution |
|----------|----------|
| Need RAG with citations | Knowledge Bases + RetrieveAndGenerate |
| Block harmful content | Guardrails with Content Filters |
| Detect hallucinations | Contextual Grounding Check |
| Autonomous task execution | Agents with Action Groups |
| Reduce model costs | Distillation (smaller student) |
| Domain-specific knowledge | Continued Pre-Training |
| Visual workflow | Bedrock Flows |
| Prompt versioning | Prompt Management |

---

*Generated from official AWS Bedrock documentation*
