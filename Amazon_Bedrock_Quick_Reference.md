# Amazon Bedrock Quick Reference & Examples

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         AMAZON BEDROCK                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐               │
│  │   Anthropic  │    │  Amazon Nova │    │     Meta     │               │
│  │    Claude    │    │  Premier/Pro │    │    Llama     │               │
│  └──────────────┘    └──────────────┘    └──────────────┘               │
│                                                                          │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐               │
│  │   Mistral    │    │   Cohere     │    │  Stability   │               │
│  │    AI        │    │  Command/Embed│   │     AI       │               │
│  └──────────────┘    └──────────────┘    └──────────────┘               │
│                                                                          │
│         ▼                   ▼                    ▼                        │
│  ┌─────────────────────────────────────────────────────────────┐        │
│  │                    UNIFIED API LAYER                         │        │
│  │     InvokeModel  |  Converse  |  ConverseStream             │        │
│  └─────────────────────────────────────────────────────────────┘        │
│                                                                          │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐       │
│  │  Knowledge  │ │   Agents    │ │  Guardrails │ │    Flows    │       │
│  │    Bases    │ │             │ │             │ │             │       │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Model Selection Decision Tree

```
                        What's your use case?
                               │
           ┌───────────────────┼───────────────────┐
           ▼                   ▼                   ▼
      Text/Chat           Image/Video         Embeddings
           │                   │                   │
     ┌─────┴─────┐       ┌─────┴─────┐            │
     ▼           ▼       ▼           ▼            ▼
 Complex    Simple   Generate     Edit      Cohere Embed
 Reasoning  Tasks    Images      Video      Titan Embed
     │           │       │           │
     ▼           ▼       ▼           ▼
 Claude     Claude   Stability  Nova Reel
 Opus/      Haiku/   AI/Nova    Luma Ray
 Sonnet     Micro    Canvas
```

---

## API Comparison Table

| Feature | InvokeModel | Converse | ConverseStream |
|---------|-------------|----------|----------------|
| **Format** | Model-specific | Unified | Unified |
| **Streaming** | Separate API | No | Yes |
| **System Prompts** | Model-dependent | Built-in | Built-in |
| **Conversation History** | Manual | Built-in | Built-in |
| **Tool Use** | Model-dependent | Built-in | Built-in |
| **Cross-Model** | No | Yes | Yes |
| **Recommended** | Legacy/specific needs | Yes | Yes (real-time) |

---

## Code Examples

### Example 1: Basic InvokeModel (Python/Boto3)

```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

# Model-specific body for Claude
body = json.dumps({
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 1024,
    "messages": [
        {"role": "user", "content": "What is machine learning?"}
    ]
})

response = bedrock.invoke_model(
    modelId="anthropic.claude-3-sonnet-20240229-v1:0",
    body=body
)

result = json.loads(response['body'].read())
print(result['content'][0]['text'])
```

### Example 2: Converse API (Recommended)

```python
import boto3

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

response = bedrock.converse(
    modelId="anthropic.claude-3-sonnet-20240229-v1:0",
    messages=[
        {
            "role": "user",
            "content": [{"text": "What is machine learning?"}]
        }
    ],
    system=[{"text": "You are a helpful AI assistant."}],
    inferenceConfig={
        "maxTokens": 1024,
        "temperature": 0.7
    }
)

print(response['output']['message']['content'][0]['text'])
```

### Example 3: Streaming Response

```python
import boto3

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

response = bedrock.converse_stream(
    modelId="anthropic.claude-3-haiku-20240307-v1:0",
    messages=[
        {"role": "user", "content": [{"text": "Write a short poem"}]}
    ]
)

# Process streaming response
for event in response['stream']:
    if 'contentBlockDelta' in event:
        print(event['contentBlockDelta']['delta']['text'], end='')
```

### Example 4: Knowledge Base Query

```python
import boto3

bedrock_agent = boto3.client('bedrock-agent-runtime', region_name='us-east-1')

response = bedrock_agent.retrieve_and_generate(
    input={"text": "What are our company's vacation policies?"},
    retrieveAndGenerateConfiguration={
        "type": "KNOWLEDGE_BASE",
        "knowledgeBaseConfiguration": {
            "knowledgeBaseId": "KB12345678",
            "modelArn": "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet-20240229-v1:0"
        }
    }
)

print(response['output']['text'])
# Citations available in response['citations']
```

### Example 5: Invoke Agent

```python
import boto3

bedrock_agent = boto3.client('bedrock-agent-runtime', region_name='us-east-1')

response = bedrock_agent.invoke_agent(
    agentId="AGENT123",
    agentAliasId="ALIAS123",
    sessionId="session-001",
    inputText="Book a flight from NYC to LA for next Monday"
)

# Process streaming response
for event in response['completion']:
    if 'chunk' in event:
        print(event['chunk']['bytes'].decode())
```

### Example 6: Apply Guardrails

```python
import boto3

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

response = bedrock.converse(
    modelId="anthropic.claude-3-sonnet-20240229-v1:0",
    messages=[
        {"role": "user", "content": [{"text": "User input here"}]}
    ],
    guardrailConfig={
        "guardrailIdentifier": "guardrail-id",
        "guardrailVersion": "1"
    }
)

# Check if guardrail was triggered
if response.get('stopReason') == 'guardrail_intervened':
    print("Content blocked by guardrail")
```

---

## RAG Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    KNOWLEDGE BASE SETUP                          │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
  ┌──────────┐         ┌──────────┐         ┌──────────┐
  │   S3     │         │  Web     │         │ Database │
  │ Documents│         │ Crawlers │         │          │
  └────┬─────┘         └────┬─────┘         └────┬─────┘
       │                    │                    │
       └────────────────────┼────────────────────┘
                            ▼
                    ┌──────────────┐
                    │   Chunking   │  ← Split into smaller pieces
                    │  Strategy    │    (512-1024 tokens typical)
                    └──────┬───────┘
                           ▼
                    ┌──────────────┐
                    │  Embedding   │  ← Titan Embed / Cohere Embed
                    │    Model     │
                    └──────┬───────┘
                           ▼
                    ┌──────────────┐
                    │ Vector Store │  ← OpenSearch / Pinecone / etc.
                    │              │
                    └──────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    QUERY TIME (RAG)                              │
└─────────────────────────────────────────────────────────────────┘

  User Query
      │
      ▼
┌──────────────┐
│  Embedding   │  ← Same model as indexing
│    Model     │
└──────┬───────┘
       │
       ▼
┌──────────────┐      ┌──────────────┐
│ Vector Store │─────>│  Top K       │  ← Retrieve relevant chunks
│   Search     │      │  Results     │
└──────────────┘      └──────┬───────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │  Augmented       │
                    │  Prompt          │
                    │                  │
                    │  Context: [docs] │
                    │  Query: [user]   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────┐
                    │     LLM      │  ← Claude / Nova / Llama
                    │              │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   Response   │  + Citations
                    │              │
                    └──────────────┘
```

---

## Agent Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     BEDROCK AGENT                                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATION LOOP                            │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                                                          │    │
│  │   1. UNDERSTAND    2. PLAN       3. EXECUTE    4. RESPOND│   │
│  │   Parse user   →  Break into  →  Call APIs  →  Generate │    │
│  │   request         steps          & KBs         response  │    │
│  │                                                          │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
       │                    │                    │
       ▼                    ▼                    ▼
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Action    │      │  Knowledge  │      │    User     │
│   Groups    │      │    Base     │      │  Response   │
│             │      │             │      │             │
│ - API 1     │      │ - RAG Query │      │ - Summary   │
│ - API 2     │      │ - Citations │      │ - Actions   │
│ - Lambda    │      │             │      │   Taken     │
└─────────────┘      └─────────────┘      └─────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ACTION GROUP DEFINITION                       │
│  OpenAPI Schema / Lambda Function                                │
│                                                                  │
│  {                                                               │
│    "openapi": "3.0.0",                                          │
│    "paths": {                                                    │
│      "/bookFlight": {                                           │
│        "post": {                                                │
│          "description": "Book a flight",                        │
│          "parameters": [...]                                    │
│        }                                                        │
│      }                                                          │
│    }                                                            │
│  }                                                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Guardrails Configuration

### Policy Types & Examples

| Policy | What It Blocks | Example |
|--------|---------------|---------|
| **Content Filter** | Harmful categories | Block hate speech, violence |
| **Denied Topics** | Specific subjects | Block investment advice |
| **Word Filter** | Exact matches | Block competitor names |
| **PII Filter** | Personal data | Mask SSN: `XXX-XX-1234` |
| **Grounding Check** | Hallucinations | Verify against source |

### Content Filter Strengths

```
NONE ──────── LOW ──────── MEDIUM ──────── HIGH
  │            │              │              │
  │            │              │              │
  ▼            ▼              ▼              ▼
No filter   Light       Moderate        Strict
            filtering   filtering      filtering
```

### Guardrail Flow

```
┌──────────┐    ┌────────────┐    ┌──────────┐    ┌────────────┐
│  User    │───>│  INPUT     │───>│   LLM    │───>│  OUTPUT    │
│  Input   │    │ GUARDRAIL  │    │          │    │ GUARDRAIL  │
└──────────┘    └────────────┘    └──────────┘    └────────────┘
                     │                                  │
                     ▼                                  ▼
               ┌──────────┐                      ┌──────────┐
               │ BLOCKED? │                      │ BLOCKED? │
               │ Return   │                      │ Return   │
               │ custom   │                      │ custom   │
               │ message  │                      │ message  │
               └──────────┘                      └──────────┘
```

---

## Model Customization Comparison

| Method | Input Data | Use Case | Output |
|--------|-----------|----------|--------|
| **Distillation** | Teacher model responses | Smaller, faster model | Student model |
| **Fine-tuning** | Labeled pairs (input→output) | Task-specific | Custom model |
| **Continued Pre-training** | Unlabeled domain text | Domain knowledge | Custom model |
| **Reinforcement** | Reward functions (Lambda) | Alignment | Custom model |

### Fine-tuning Data Format (JSONL)

```jsonl
{"prompt": "Summarize this article:", "completion": "The article discusses..."}
{"prompt": "Translate to French:", "completion": "Bonjour, comment..."}
{"prompt": "Extract entities:", "completion": "Person: John, Location: NYC"}
```

### Distillation Flow

```
┌───────────────┐         ┌───────────────┐
│    TEACHER    │         │    STUDENT    │
│  (Large Model)│         │ (Small Model) │
│               │         │               │
│  Claude Opus  │         │  Claude Haiku │
│  Nova Premier │         │  Nova Micro   │
└───────┬───────┘         └───────┬───────┘
        │                         │
        │    Your Prompts         │
        │         │               │
        ▼         ▼               │
   ┌─────────────────┐            │
   │ Teacher generates│           │
   │ high-quality     │           │
   │ responses        │           │
   └────────┬────────┘            │
            │                     │
            ▼                     ▼
   ┌─────────────────────────────────┐
   │    Fine-tune student model      │
   │    on teacher's responses       │
   └─────────────────────────────────┘
                    │
                    ▼
   ┌─────────────────────────────────┐
   │   Smaller model with similar    │
   │   quality, lower cost/latency   │
   └─────────────────────────────────┘
```

---

## Pricing Models

### On-Demand vs Provisioned

| Aspect | On-Demand | Provisioned Throughput |
|--------|-----------|----------------------|
| **Billing** | Per token | Fixed hourly |
| **Best For** | Variable workloads | Consistent high-volume |
| **Custom Models** | Not supported | Required |
| **Commitment** | None | None / 1mo / 6mo |
| **Discount** | None | Up to ~50% |

### Token Pricing Factors

```
Total Cost = Input Tokens × Input Price + Output Tokens × Output Price
```

| Model Tier | Input (per 1K) | Output (per 1K) |
|------------|---------------|-----------------|
| Haiku/Micro | $0.00025 | $0.00125 |
| Sonnet/Pro | $0.003 | $0.015 |
| Opus/Premier | $0.015 | $0.075 |

*(Prices approximate - check AWS for current pricing)*

---

## Flows Node Types

```
┌─────────────────────────────────────────────────────────────────┐
│                      FLOW EXAMPLE                                │
│                                                                  │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐     │
│   │  INPUT  │───>│ PROMPT  │───>│   KB    │───>│ LAMBDA  │     │
│   │  Node   │    │  Node   │    │  Node   │    │  Node   │     │
│   └─────────┘    └─────────┘    └─────────┘    └─────────┘     │
│                                                      │          │
│                                                      ▼          │
│                                               ┌─────────┐       │
│                                               │ OUTPUT  │       │
│                                               │  Node   │       │
│                                               └─────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

| Node Type | Purpose | Example |
|-----------|---------|---------|
| **Input** | Receive user input | Start of flow |
| **Prompt** | Execute FM with prompt | Generate summary |
| **Knowledge Base** | Query KB for RAG | Retrieve docs |
| **Lambda** | Execute custom code | Send email |
| **Condition** | Branch logic | If/else |
| **Output** | Return response | End of flow |

---

## IAM Policy Examples

### Basic Bedrock Access

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": "arn:aws:bedrock:*::foundation-model/*"
        }
    ]
}
```

### Specific Model Access

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["bedrock:InvokeModel"],
            "Resource": [
                "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet*",
                "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-haiku*"
            ]
        }
    ]
}
```

### Knowledge Base Access

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:Retrieve",
                "bedrock:RetrieveAndGenerate"
            ],
            "Resource": "arn:aws:bedrock:us-east-1:123456789012:knowledge-base/*"
        }
    ]
}
```

---

## Common Exam Scenarios

### Scenario 1: Choose the Right API
> "Application needs to work with multiple models and support conversation history"

**Answer**: Use **Converse API** (unified interface, built-in conversation support)

### Scenario 2: Reduce Hallucinations
> "Company wants to ensure AI responses are grounded in their documentation"

**Answer**: Use **Knowledge Base with RAG** + **Guardrails with Grounding Checks**

### Scenario 3: Block Sensitive Content
> "Banking app must not provide investment advice"

**Answer**: Configure **Guardrails with Denied Topics** for investment-related queries

### Scenario 4: Cost Optimization
> "Startup has variable traffic, sometimes 1000 requests/day, sometimes 10"

**Answer**: Use **On-Demand pricing** (no commitment, pay per use)

### Scenario 5: Custom Model Deployment
> "After fine-tuning, how to use the custom model?"

**Answer**: Must purchase **Provisioned Throughput** for custom models

### Scenario 6: Real-time Chat Application
> "Building a chatbot that needs to show responses as they're generated"

**Answer**: Use **ConverseStream** API for streaming responses

### Scenario 7: Autonomous Task Execution
> "System needs to query databases and call external APIs based on user requests"

**Answer**: Use **Bedrock Agents** with **Action Groups** defining the APIs

---

## Quick Facts for Exam

### Must Remember
- [ ] **Converse API** = unified, recommended for most use cases
- [ ] **InvokeModel** = model-specific, legacy approach
- [ ] **Agents require Action Groups** (at least one)
- [ ] **Custom models require Provisioned Throughput**
- [ ] **Knowledge Bases enable RAG**
- [ ] **Guardrails** apply to both input AND output
- [ ] **Flows** = visual workflow builder

### Model Categories
- [ ] **Text**: Claude, Nova, Llama, Mistral, Command
- [ ] **Image Gen**: Stability AI, Nova Canvas, Titan Image
- [ ] **Embeddings**: Titan Embed, Cohere Embed
- [ ] **Video**: Nova Reel
- [ ] **Multimodal Input**: Claude, Nova, Llama 3.2+

### Security Checklist
- [ ] IAM policies for model access
- [ ] VPC endpoints for private access
- [ ] KMS for encryption
- [ ] CloudTrail for audit logging
- [ ] Guardrails for content safety
