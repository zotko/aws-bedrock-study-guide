# Amazon Bedrock Study Guide for AWS AI Practitioner Exam

## What is Amazon Bedrock?

**Fully managed, serverless service** providing access to foundation models (FMs) from leading AI companies through a **unified API**.

**Key Characteristics:**
- No infrastructure management
- Single API for multiple models
- Built-in security, privacy, and responsible AI

---

## Foundation Models Available

### Model Providers & Key Models

| Provider | Models | Best For |
|----------|--------|----------|
| **Anthropic** | Claude Opus 4.5, Sonnet 4.5, Haiku | General AI, reasoning, code |
| **Amazon Nova** | Premier, Pro, Lite, Micro, Canvas, Reel, Sonic | Multimodal, video, image, speech |
| **Meta** | Llama 4, Llama 3.3, Llama 3.2, Llama 3.1 | Instruction-following, large-scale |
| **Mistral** | Pixtral Large, Mistral Large, Ministral | Fast inference, vision |
| **Cohere** | Command R+, Command R, Embed, Rerank | Search, embeddings, ranking |
| **Stability AI** | SD 3.5, Stable Image Ultra/Inpaint/Outpaint | Image generation/editing |
| **Amazon Titan** | Text, Embeddings G1/V2, Image Generator | Text, embeddings, images |
| **Google** | Gemma 3 (27B, 12B, 4B) | Lightweight inference |

### Input/Output Modalities

| Modality | Input | Output |
|----------|-------|--------|
| **Text** | Most models | Most models |
| **Image** | Claude, Nova, Llama 3.2+, Mistral Pixtral | Stability, Titan, Nova Canvas |
| **Video** | Nova Premier/Pro/Lite | Nova Reel, Luma Ray |
| **Audio** | Nova Sonic | Nova Sonic |
| **Embeddings** | Text | Titan Embed, Cohere Embed |

---

## Core APIs

### InvokeModel API
```
Purpose: Submit prompt, get response
Streaming: InvokeModelWithResponseStream
Format: Model-specific request body
```

### Converse API (Recommended)
```
Purpose: Unified interface across ALL models
Features:
  - Model-agnostic format
  - System prompts
  - Conversation history
  - Tool use support
Streaming: ConverseStream
```

### Other APIs
- **StartAsyncInvoke**: Async generation (video)
- **InvokeModelWithBidirectionalStream**: Bidirectional streaming
- **OpenAI Chat Completions**: OpenAI-compatible interface

### Model ID Types
| Type | Description |
|------|-------------|
| Base Model | Foundation model from provider |
| Inference Profile | Cross-region throughput |
| Provisioned Throughput | Fixed-cost higher throughput |
| Custom Model | Fine-tuned model (requires provisioned) |
| Prompt | Reusable prompt from Prompt Management |

---

## Knowledge Bases (RAG)

### What They Do
Enable **Retrieval Augmented Generation** - integrate proprietary data with foundation models.

### Key Features
- Data retrieval from custom sources
- Response augmentation with enterprise data
- **Citations** for source verification
- Multimodal content support
- Structured data queries (natural language → SQL)
- Real-time sync with data sources
- Reranking support

### Setup Process
1. Set up vector store (or use OpenSearch Serverless auto-created)
2. Connect data source (structured or unstructured)
3. Sync data
4. Query and generate responses

### Data Source Types
- **Unstructured**: Documents, text, images (requires vector store)
- **Structured**: Databases (SQL conversion)
- **Specialized**: Kendra GenAI, Neptune Analytics

---

## Agents

### What They Do
Autonomous agents that orchestrate between FMs, data sources, and applications.

### Core Capabilities
1. **Understand & Decompose** - Break down user requests
2. **Collect Information** - Gather details via conversation
3. **Take Actions** - Make API calls to systems
4. **Augment Performance** - Query knowledge bases

### Components

| Component | Required | Purpose |
|-----------|----------|---------|
| **Action Groups** | Yes | Define actions agent can perform |
| **Knowledge Bases** | No | Store organizational data for augmentation |

### Implementation Flow
```
Create KB (optional) → Configure Agent + Action Groups →
Customize Prompts → Test (TSTALIASID) → Create Alias →
Deploy → Integrate via API
```

### Managed By AWS
- Prompt engineering
- Memory management
- Monitoring & logging
- Encryption
- User permissions
- API invocation

---

## Guardrails

### Purpose
Configurable safeguards to detect and filter harmful content.

### Safeguard Types

| Type | Function |
|------|----------|
| **Content Filters** | Detect harmful content (hate, violence, sexual, etc.) |
| **Denied Topics** | Block specific topics |
| **Word Filters** | Block exact words/phrases/profanity |
| **Sensitive Info Filters** | Block/mask PII, custom regex |
| **Contextual Grounding** | Detect hallucinations based on source |
| **Automated Reasoning** | Validate against logical rules |

### Content Filter Categories
- Hate
- Insults
- Sexual
- Violence
- Misconduct
- Prompt Attack

### Implementation
- Specify guardrail ID + version in API calls
- Or use `ApplyGuardrail` API independently
- Configure custom violation messages
- Versioning for iterative development

---

## Model Customization

### Methods

| Method | Data Type | Purpose |
|--------|-----------|---------|
| **Distillation** | Teacher model output | Transfer knowledge to smaller model |
| **Reinforcement Fine-tuning** | Reward functions | Align model with business requirements |
| **Supervised Fine-tuning** | Labeled data | Task-specific performance |
| **Continued Pre-training** | Unlabeled data | Domain familiarization |

### Distillation Process
1. Select teacher model (high accuracy)
2. Select student model (to fine-tune)
3. Provide use-case prompts
4. Bedrock generates teacher responses
5. Student model fine-tuned on those responses

### Pricing
- **Training**: Per tokens processed (tokens × epochs)
- **Storage**: Per-month per custom model
- **Inference**: Requires Provisioned Throughput

---

## Provisioned Throughput

### What It Is
Purchase higher throughput at **fixed hourly cost** vs. per-request pricing.

### Model Units (MU)
Each MU specifies:
- Input tokens processable per minute
- Output tokens generatable per minute

### Commitment Options

| Duration | Discount | Flexibility |
|----------|----------|-------------|
| No commitment | None | Delete anytime |
| 1 month | Moderate | Cannot delete until term ends |
| 6 months | Maximum | Cannot delete until term ends |

### When to Use
- Consistent, high-volume workloads
- Custom models (required)
- Predictable capacity needs

---

## Prompt Management

### Features
- Create and store reusable prompts
- **Variables**: Placeholders for dynamic values
- **Variants**: Alternative configurations to compare
- **Prompt Builder**: Visual console tool

### Workflow
1. Create prompt with variables
2. Select model and configure settings
3. Test with sample values
4. Create and compare variants
5. Integrate via inference or Flows

---

## Bedrock Flows

### What They Do
Build end-to-end GenAI workflows connecting FMs, prompts, and AWS services.

### Key Features
- Visual drag-and-drop builder
- API management
- Connect FMs, KBs, Lambda
- Immutable versioned deployments

### Workflow Process
1. **Create**: Define flow, nodes, connections
2. **Test**: Prepare draft, invoke with samples
3. **Deploy**: Publish version, create alias, invoke

### Example Use Cases
- Email invitations
- Error troubleshooting
- Report generation
- Data ingestion pipelines

---

## Security

### Shared Responsibility

| AWS Responsibility | Customer Responsibility |
|--------------------|------------------------|
| Infrastructure | Data sensitivity |
| Data center security | Company requirements |
| Third-party audits | Compliance |

### Key Security Features

| Feature | Description |
|---------|-------------|
| **Data Protection** | Encryption at rest and in transit |
| **IAM** | Role-based access control |
| **Cross-Account** | S3 access for custom models |
| **VPC** | Private connectivity |
| **Confused Deputy Prevention** | Cross-service authorization |
| **Prompt Injection Prevention** | LLM-specific attack protection |

---

## Key Exam Concepts

### Remember These
1. **Converse API** = unified interface across models
2. **InvokeModel** = model-specific interface
3. **Knowledge Bases** = RAG implementation
4. **Agents** = autonomous task execution
5. **Guardrails** = content filtering & safety
6. **Provisioned Throughput** = required for custom models
7. **Action Groups** = required for agents

### Common Patterns

**RAG Pattern:**
```
User Query → Embedding → Vector Search →
Retrieved Context + Query → LLM → Response with Citations
```

**Agent Pattern:**
```
User Request → Agent Decomposition →
Action Group API Calls + KB Queries →
Orchestrated Response
```

### Cost Optimization
- Choose appropriate model size
- Use Provisioned Throughput for high volume
- Implement prompt caching
- Use smaller models (Haiku, Micro) for simple tasks
- Batch similar requests

### Model Selection Guide

| Use Case | Recommended |
|----------|-------------|
| Complex reasoning | Claude Opus, Nova Premier |
| Fast responses | Claude Haiku, Nova Micro |
| Image generation | Stability AI, Nova Canvas |
| Embeddings | Titan Embed, Cohere Embed |
| Video generation | Nova Reel |
| Code generation | Claude, Llama |

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────┐
│                 AMAZON BEDROCK                       │
├─────────────────────────────────────────────────────┤
│ APIs                                                 │
│   InvokeModel / InvokeModelWithResponseStream       │
│   Converse / ConverseStream (unified, recommended)  │
│   StartAsyncInvoke (async, video)                   │
├─────────────────────────────────────────────────────┤
│ Features                                             │
│   Knowledge Bases → RAG                              │
│   Agents → Autonomous tasks                          │
│   Guardrails → Safety & content filtering            │
│   Flows → Workflow orchestration                     │
│   Prompt Management → Reusable prompts               │
├─────────────────────────────────────────────────────┤
│ Customization                                        │
│   Distillation → Teacher → Student                   │
│   Fine-tuning → Labeled data                         │
│   Continued Pre-training → Unlabeled data            │
│   Reinforcement → Reward functions                   │
├─────────────────────────────────────────────────────┤
│ Pricing Models                                       │
│   On-Demand → Per token                              │
│   Provisioned → Fixed hourly (required for custom)   │
└─────────────────────────────────────────────────────┘
```
