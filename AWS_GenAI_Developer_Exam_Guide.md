# AWS Certified AI Practitioner (AIF-C01) Exam Preparation Guide

## Exam Overview

| Attribute | Details |
|-----------|---------|
| **Exam Code** | AIF-C01 |
| **Total Questions** | 75 (65 scored + 10 unscored) |
| **Score Range** | 100-1,000 (scaled) |
| **Passing Score** | 750 |
| **Time** | 120 minutes |

## Question Types

- **Multiple Choice**: 1 correct answer from 4 options
- **Multiple Response**: 2+ correct answers from 5+ options
- **Ordering**: Arrange 3-5 responses in correct sequence
- **Matching**: Match responses to 3-7 prompts

**Note**: No penalty for guessing. Unanswered questions count as incorrect.

---

## Content Domains & Weightings

### Domain 1: Foundation Model Integration, Data Management, and Compliance (31%)

**Focus Areas:**
- Vector stores and embeddings
- Retrieval-Augmented Generation (RAG)
- Knowledge bases
- GenAI architectures
- Data management for AI
- Compliance requirements

**Key Concepts:**
- Embedding models and vector databases
- Chunking strategies for documents
- Semantic search implementation
- Data preprocessing for LLMs
- Regulatory compliance (GDPR, HIPAA considerations)

---

### Domain 2: Implementation and Integration (26%)

**Focus Areas:**
- Foundation model integration into applications
- Prompt engineering techniques
- Prompt management
- Agentic AI solutions

**Key Concepts:**
- Amazon Bedrock integration
- API calls to foundation models
- Zero-shot, few-shot, chain-of-thought prompting
- Prompt templates and versioning
- Agent architectures and tool use
- Orchestration patterns

---

### Domain 3: AI Safety, Security, and Governance (20%)

**Focus Areas:**
- Security best practices
- Governance frameworks
- Responsible AI practices

**Key Concepts:**
- IAM policies for AI services
- Data encryption (at rest and in transit)
- Model access controls
- Guardrails implementation
- Bias detection and mitigation
- Content filtering
- Audit logging and compliance

---

### Domain 4: Operational Efficiency and Optimization (12%)

**Focus Areas:**
- Cost optimization
- Performance optimization
- Business value measurement

**Key Concepts:**
- Token usage optimization
- Model selection for cost/performance trade-offs
- Caching strategies
- Latency optimization
- Throughput management
- ROI measurement

---

### Domain 5: Testing, Validation, and Troubleshooting (11%)

**Focus Areas:**
- Troubleshooting GenAI applications
- Monitoring and observability
- Optimization techniques
- Foundation model evaluation

**Key Concepts:**
- Evaluation metrics (BLEU, ROUGE, human evaluation)
- A/B testing for prompts
- Hallucination detection
- Response quality assessment
- CloudWatch integration
- Debugging common issues

---

## Key AWS Services to Study

### Primary Services

| Service | Purpose |
|---------|---------|
| **Amazon Bedrock** | Managed service for foundation models |
| **Amazon SageMaker** | ML model training and deployment |
| **Amazon Q** | AI-powered assistant |
| **Amazon Kendra** | Intelligent search service |
| **Amazon Comprehend** | NLP service |
| **Amazon Textract** | Document text extraction |
| **Amazon Transcribe** | Speech-to-text |
| **Amazon Polly** | Text-to-speech |
| **Amazon Rekognition** | Image and video analysis |
| **Amazon Translate** | Language translation |

### Supporting Services

| Service | Purpose |
|---------|---------|
| **Amazon S3** | Data storage |
| **Amazon OpenSearch** | Vector search capabilities |
| **AWS Lambda** | Serverless compute for AI apps |
| **Amazon API Gateway** | API management |
| **AWS IAM** | Access management |
| **Amazon CloudWatch** | Monitoring and logging |
| **AWS CloudTrail** | Audit logging |
| **AWS KMS** | Encryption key management |

---

## Amazon Bedrock Deep Dive

### Foundation Models Available
- Amazon Titan (Text, Embeddings, Image)
- Anthropic Claude
- AI21 Labs Jurassic
- Cohere Command
- Meta Llama
- Stability AI

### Key Features
- **Knowledge Bases**: RAG implementation
- **Agents**: Autonomous task execution
- **Guardrails**: Content filtering and safety
- **Fine-tuning**: Custom model adaptation
- **Provisioned Throughput**: Dedicated capacity

---

## RAG Architecture Components

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Documents  │───>│  Chunking   │───>│  Embedding  │
└─────────────┘    └─────────────┘    └─────────────┘
                                             │
                                             v
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Response  │<───│     LLM     │<───│Vector Store │
└─────────────┘    └─────────────┘    └─────────────┘
                          ^
                          │
                   ┌─────────────┐
                   │ User Query  │
                   └─────────────┘
```

### Key Considerations
- Chunk size optimization (typically 512-1024 tokens)
- Overlap between chunks
- Metadata preservation
- Embedding model selection
- Vector database indexing strategies

---

## Prompt Engineering Techniques

### Basic Techniques
1. **Zero-shot**: Direct question without examples
2. **Few-shot**: Provide examples before query
3. **Chain-of-thought**: Step-by-step reasoning

### Advanced Techniques
1. **Self-consistency**: Multiple reasoning paths
2. **Tree of thoughts**: Branching reasoning
3. **ReAct**: Reasoning + Acting pattern

### Best Practices
- Be specific and clear
- Provide context
- Use delimiters for structure
- Specify output format
- Include constraints

---

## Security Best Practices

### Data Protection
- Encrypt data at rest (S3 SSE, KMS)
- Encrypt data in transit (TLS)
- Implement VPC endpoints for private access
- Use AWS PrivateLink

### Access Control
- Least privilege IAM policies
- Resource-based policies
- Service control policies (SCPs)
- Cross-account access management

### Monitoring
- CloudWatch Logs for API calls
- CloudTrail for audit
- AWS Config for compliance
- Security Hub integration

---

## Cost Optimization Strategies

1. **Model Selection**: Choose appropriate model size
2. **Token Optimization**: Minimize input/output tokens
3. **Caching**: Cache common responses
4. **Batch Processing**: Group similar requests
5. **Provisioned Throughput**: For predictable workloads
6. **On-Demand**: For variable workloads

---

## Evaluation Metrics

### Automated Metrics
- **BLEU**: Translation quality
- **ROUGE**: Summarization quality
- **Perplexity**: Language model quality
- **F1 Score**: Classification accuracy

### Human Evaluation
- Relevance
- Coherence
- Fluency
- Factual accuracy
- Helpfulness

---

## Exam Preparation Tips

1. **Hands-on Practice**: Use AWS Free Tier for Bedrock experimentation
2. **Study Documentation**: AWS Bedrock and SageMaker docs
3. **Practice Exams**: AWS Skill Builder practice tests
4. **Focus on Domains**: Prioritize Domain 1 (31%) and Domain 2 (26%)
5. **Understand Trade-offs**: Cost vs. performance, latency vs. accuracy

---

## Out of Scope Topics

- Model development and training from scratch
- Advanced ML techniques (backpropagation, etc.)
- Data engineering and feature engineering
- Deep mathematical foundations

---

## Additional Resources

- [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [AWS Skill Builder](https://skillbuilder.aws/)
- [AWS Certification Portal](https://aws.amazon.com/certification/)
- [AWS Well-Architected Framework - ML Lens](https://docs.aws.amazon.com/wellarchitected/latest/machine-learning-lens/)
