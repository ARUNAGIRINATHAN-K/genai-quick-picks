# Cohere SDK Reference

**Latest Update**: June 2026  
**Official Docs**: [docs.cohere.com](https://docs.cohere.com)  
**GitHub**: [cohere-ai/cohere-python](https://github.com/cohere-ai/cohere-python), [cohere-ai/cohere-typescript](https://github.com/cohere-ai/cohere-typescript)

## Table of Contents

- [Installation](#installation)
- [SDK Initialization](#sdk-initialization)
- [Authentication](#authentication)
- [Chat API (Core Generation)](#chat-api-core-generation)
- [Available Models](#available-models)
- [Embeddings API](#embeddings-api)
- [Rerank API](#rerank-api)
- [Tool Use (Function Calling)](#tool-use-function-calling)
- [Structured Outputs (JSON Mode)](#structured-outputs-json-mode)
- [Tokenization](#tokenization)
- [Fine-tuning](#fine-tuning)
- [Async Operations](#async-operations)
- [Error Handling](#error-handling)
- [Compatibility API (OpenAI SDK)](#compatibility-api-openai-sdk)
- [Production Best Practices](#production-best-practices)
- [Quick Reference](#quick-reference)
- [Links & Resources](#links--resources)

---

## Installation

### Python
```bash
pip install cohere --upgrade
# Requires Python 3.7+
```

### TypeScript/JavaScript
```bash
npm install cohere-ai
# Requires Node.js 14+
```

---

## SDK Initialization

### Python
```python
import cohere

# Basic initialization
co = cohere.ClientV2(api_key="your-api-key")

# Or use environment variable
# export CO_API_KEY="your-api-key"
co = cohere.ClientV2()

# With custom timeout
co = cohere.ClientV2(
    api_key="your-api-key",
    timeout=30.0
)
```

### Python - Cloud Deployments
```python
# AWS Bedrock
co = cohere.BedrockClient(
    aws_region="us-east-1",
    aws_access_key="...",
    aws_secret_key="..."
)

# Azure
co = cohere.AzureClient(
    azure_api_key="...",
    azure_endpoint="..."
)

# Google Cloud (Vertex AI)
co = cohere.VertexClient(
    project_id="your-project",
    region="us-central1"
)

# OCI
co = cohere.OciClient(
    oci_region="us-chicago-1",
    oci_compartment_id="ocid1.compartment.oc1..."
)
```

### TypeScript/JavaScript
```typescript
import { CohereClientV2 } from "cohere-ai";

// Basic initialization
const cohere = new CohereClientV2({
  token: "your-api-key"
});

// Or use environment variable
// export COHERE_API_KEY="your-api-key"
const cohere = new CohereClientV2();

// With custom configuration
const cohere = new CohereClientV2({
  token: process.env.COHERE_API_KEY,
  clientName: "my-app",
  timeout: 30000
});
```

### TypeScript - AWS Bedrock
```typescript
import { BedrockClient } from "cohere-ai/aws";

const cohere = new BedrockClient({
  awsRegion: "us-east-1",
  awsAccessKeyId: "...",
  awsSecretAccessKey: "..."
});
```

---

## Authentication

Cohere uses **API key authentication** via the `Authorization: Bearer` header.

```bash
# Set environment variable
export CO_API_KEY="your-api-key"
```

Get your API key from [Cohere Dashboard](https://dashboard.cohere.com/api-keys).

---

## Chat API (Core Generation)

Cohere's primary text generation interface with multi-turn conversation support.

### Python - Unary (Simple)
```python
import cohere

co = cohere.ClientV2()

response = co.chat(
    model="command-a-03-2025",
    messages=[
        {
            "role": "user",
            "content": "Explain machine learning in simple terms"
        }
    ]
)

print(response.message.content[0].text)
```

### Python - Streaming
```python
stream = co.chat_stream(
    model="command-a-03-2025",
    messages=[
        {
            "role": "user",
            "content": "Write a short story about AI"
        }
    ]
)

for event in stream:
    if hasattr(event, 'delta') and hasattr(event.delta, 'message'):
        print(event.delta.message.content[0].text, end="")
```

### Python - Multi-turn Conversation
```python
messages = [
    {"role": "user", "content": "What is AI?"},
]

# First turn
response1 = co.chat(
    model="command-a-03-2025",
    messages=messages
)
print("Assistant:", response1.message.content[0].text)

# Add assistant response to history
messages.append({
    "role": "assistant",
    "content": response1.message.content[0].text
})

# Second turn
messages.append({
    "role": "user",
    "content": "Can you elaborate on that?"
})

response2 = co.chat(
    model="command-a-03-2025",
    messages=messages
)
print("Assistant:", response2.message.content[0].text)
```

### TypeScript - Unary
```typescript
import { CohereClientV2 } from "cohere-ai";

const cohere = new CohereClientV2();

const response = await cohere.chat({
  model: "command-a-03-2025",
  messages: [
    {
      role: "user",
      content: "Explain machine learning in simple terms"
    }
  ]
});

console.log(response.message.content[0].text);
```

### TypeScript - Streaming
```typescript
const stream = await cohere.chatStream({
  model: "command-a-03-2025",
  messages: [
    {
      role: "user",
      content: "Write a short story about AI"
    }
  ]
});

for await (const event of stream) {
  if (event.type === "content-delta" && event.delta?.message) {
    process.stdout.write(event.delta.message.content[0].text);
  }
}
```

### Common Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `model` | string | Model ID (see below) |
| `messages` | array | Message history with `role` and `content` |
| `temperature` | float | 0.0-2.0; higher = more creative |
| `max_tokens` | int | Maximum response length |
| `p` | float | Nucleus sampling (0-1) |
| `k` | int | Top-K sampling |
| `frequency_penalty` | float | Penalize repetition |
| `presence_penalty` | float | Encourage new topics |
| `seed` | int | Reproducibility |
| `stop_sequences` | list | Stop generation at these tokens |

---

## Available Models

| Model | Context | Use Case |
|-------|---------|----------|
| `command-a-03-2025` | 128K | Latest general-purpose, best quality |
| `command-r-plus` | 128K | Advanced reasoning, RAG, tool use |
| `command-r` | 128K | Fast balanced performance |
| `command-light` | 4K | Ultra-fast, low-latency |
| `aya-23-35b` | 4K | Multilingual (23+ languages) |

---

## Embeddings API

Generate vector embeddings for semantic search, clustering, and RAG.

### Python
```python
import cohere

co = cohere.ClientV2()

# Single text
response = co.embed(
    model="embed-english-v3.0",
    input_type="search_document",  # or "search_query"
    texts=["The quick brown fox jumps over the lazy dog"]
)

print(f"Embedding dimension: {len(response.embeddings[0])}")
print(f"First 5 values: {response.embeddings[0][:5]}")

# Multiple texts
response = co.embed(
    model="embed-english-v3.0",
    input_type="search_document",
    texts=[
        "Text 1",
        "Text 2",
        "Text 3"
    ]
)

for i, embedding in enumerate(response.embeddings):
    print(f"Text {i}: {len(embedding)} dimensions")
```

### TypeScript
```typescript
const response = await cohere.embed({
  model: "embed-english-v3.0",
  inputType: "search_document",
  texts: ["The quick brown fox jumps over the lazy dog"]
});

console.log(`Embedding dimension: ${response.embeddings[0].length}`);
console.log(`First 5 values: ${response.embeddings[0].slice(0, 5)}`);

// Batch
const batchResponse = await cohere.embed({
  model: "embed-english-v3.0",
  inputType: "search_document",
  texts: ["Text 1", "Text 2", "Text 3"]
});

batchResponse.embeddings.forEach((emb, i) => {
  console.log(`Text ${i}: ${emb.length} dimensions`);
});
```

### Embedding Types

| Model | Dimension | Use Case |
|-------|-----------|----------|
| `embed-english-v3.0` | 1024 | English semantic search |
| `embed-multilingual-v3.0` | 1024 | 100+ language support |
| `embed-english-light-v3.0` | 384 | Lightweight English |

---

## Rerank API

Re-score documents for relevance to a query.

### Python
```python
import cohere

co = cohere.ClientV2()

documents = [
    "The Eiffel Tower is located in Paris, France.",
    "The Statue of Liberty is in New York.",
    "Big Ben is in London, England."
]

response = co.rerank(
    model="rerank-english-v3.0",
    query="Where is the Eiffel Tower?",
    documents=documents,
    top_n=2
)

for result in response.results:
    print(f"Document {result.index}: {result.relevance_score:.2f}")
```

### TypeScript
```typescript
const documents = [
  "The Eiffel Tower is located in Paris, France.",
  "The Statue of Liberty is in New York.",
  "Big Ben is in London, England."
];

const response = await cohere.rerank({
  model: "rerank-english-v3.0",
  query: "Where is the Eiffel Tower?",
  documents,
  topN: 2
});

response.results.forEach((result) => {
  console.log(`Document ${result.index}: ${result.relevanceScore.toFixed(2)}`);
});
```

---

## Tool Use (Function Calling)

Enable models to call external functions and APIs.

### Python
```python
import cohere
import json

co = cohere.ClientV2()

tools = [
    {
        "name": "calculator",
        "description": "Performs mathematical calculations",
        "parameter_definitions": {
            "expression": {
                "description": "Mathematical expression to evaluate",
                "type": "str",
                "required": True
            }
        }
    }
]

response = co.chat(
    model="command-a-03-2025",
    messages=[
        {
            "role": "user",
            "content": "What is 25 * 4 + 10?"
        }
    ],
    tools=tools
)

# Check for tool calls
if response.message.tool_calls:
    for tool_call in response.message.tool_calls:
        print(f"Tool: {tool_call.name}")
        print(f"Parameters: {tool_call.parameters}")
        
        # Execute tool
        if tool_call.name == "calculator":
            result = eval(tool_call.parameters["expression"])
            
            # Continue conversation
            messages = [
                {"role": "user", "content": "What is 25 * 4 + 10?"},
                {
                    "role": "assistant",
                    "tool_calls": response.message.tool_calls
                },
                {
                    "role": "tool",
                    "content": json.dumps({"result": result}),
                    "tool_call_id": tool_call.id
                }
            ]
            
            final_response = co.chat(
                model="command-a-03-2025",
                messages=messages,
                tools=tools
            )
            print(f"Answer: {final_response.message.content[0].text}")
```

### TypeScript
```typescript
const tools = [
  {
    name: "calculator",
    description: "Performs mathematical calculations",
    parameterDefinitions: {
      expression: {
        description: "Mathematical expression to evaluate",
        type: "str",
        required: true
      }
    }
  }
];

const response = await cohere.chat({
  model: "command-a-03-2025",
  messages: [
    {
      role: "user",
      content: "What is 25 * 4 + 10?"
    }
  ],
  tools
});

if (response.message.toolCalls) {
  for (const toolCall of response.message.toolCalls) {
    console.log(`Tool: ${toolCall.name}`);
    console.log(`Parameters: ${JSON.stringify(toolCall.parameters)}`);
    
    if (toolCall.name === "calculator") {
      const result = eval(toolCall.parameters.expression);
      
      const messages = [
        { role: "user" as const, content: "What is 25 * 4 + 10?" },
        { role: "assistant" as const, toolCalls: response.message.toolCalls },
        {
          role: "tool" as const,
          content: JSON.stringify({ result }),
          toolCallId: toolCall.id
        }
      ];
      
      const finalResponse = await cohere.chat({
        model: "command-a-03-2025",
        messages,
        tools
      });
      console.log(`Answer: ${finalResponse.message.content[0].text}`);
    }
  }
}
```

---

## Structured Outputs (JSON Mode)

Force model to respond with valid JSON.

### Python
```python
response = co.chat(
    model="command-a-03-2025",
    messages=[
        {
            "role": "user",
            "content": "Extract name, age, and email from: John Doe, 28, john@example.com"
        }
    ],
    response_format={"type": "json_object"}
)

import json
data = json.loads(response.message.content[0].text)
print(f"Name: {data['name']}, Age: {data['age']}")
```

### TypeScript
```typescript
const response = await cohere.chat({
  model: "command-a-03-2025",
  messages: [
    {
      role: "user",
      content: "Extract name, age, and email from: John Doe, 28, john@example.com"
    }
  ],
  responseFormat: { type: "json_object" }
});

const data = JSON.parse(response.message.content[0].text);
console.log(`Name: ${data.name}, Age: ${data.age}`);
```

---

## Tokenization

Count tokens and manage token budgets.

### Python
```python
import cohere

co = cohere.ClientV2()

# Tokenize
tokens = co.tokenize(
    model="command-a-03-2025",
    text="The quick brown fox jumps over the lazy dog"
)

print(f"Token count: {len(tokens.tokens)}")
print(f"Tokens: {tokens.tokens}")

# Detokenize
text = co.detokenize(
    model="command-a-03-2025",
    tokens=[1234, 5678, 9012]
)

print(f"Reconstructed text: {text.text}")
```

### TypeScript
```typescript
const tokens = await cohere.tokenize({
  model: "command-a-03-2025",
  text: "The quick brown fox jumps over the lazy dog"
});

console.log(`Token count: ${tokens.tokens.length}`);
console.log(`Tokens: ${tokens.tokens}`);

const text = await cohere.detokenize({
  model: "command-a-03-2025",
  tokens: [1234, 5678, 9012]
});

console.log(`Reconstructed text: ${text.text}`);
```

---

## Fine-tuning

Train custom models on your data.

### Python
```python
import cohere

co = cohere.ClientV2()

# Upload training data (JSONL format)
with open("training_data.jsonl", "rb") as f:
    uploaded_file = co.files.upload(file=f)

# Create fine-tuning job
ft_job = co.fine_tuning.create(
    model="command-r-plus",
    training_file=uploaded_file.id,
    hyperparameters={
        "learning_rate": 0.001,
        "batch_size": 8,
        "num_epochs": 3
    }
)

print(f"Job ID: {ft_job.id}")

# Monitor job status
import time
while True:
    job = co.fine_tuning.get(ft_job.id)
    print(f"Status: {job.status}")
    
    if job.status in ["completed", "failed"]:
        break
    
    time.sleep(10)

# Use fine-tuned model
if job.status == "completed":
    response = co.chat(
        model=job.fine_tuned_model_id,
        messages=[{"role": "user", "content": "Hello"}]
    )
```

### TypeScript
```typescript
// Upload training data
const uploadedFile = await cohere.files.upload({
  file: fs.createReadStream("training_data.jsonl")
});

// Create fine-tuning job
const ftJob = await cohere.fineTuning.create({
  model: "command-r-plus",
  trainingFile: uploadedFile.id,
  hyperparameters: {
    learningRate: 0.001,
    batchSize: 8,
    numEpochs: 3
  }
});

console.log(`Job ID: ${ftJob.id}`);

// Monitor job
let job = ftJob;
while (job.status !== "completed" && job.status !== "failed") {
  await new Promise(resolve => setTimeout(resolve, 10000));
  job = await cohere.fineTuning.get(ftJob.id);
  console.log(`Status: ${job.status}`);
}

// Use fine-tuned model
if (job.status === "completed") {
  const response = await cohere.chat({
    model: job.fineTunedModelId,
    messages: [{ role: "user", content: "Hello" }]
  });
}
```

---

## Async Operations

### Python
```python
import asyncio
import cohere

async def main():
    co = cohere.AsyncClientV2(api_key="your-key")
    
    response = await co.chat(
        model="command-a-03-2025",
        messages=[{"role": "user", "content": "Hello"}]
    )
    
    print(response.message.content[0].text)

asyncio.run(main())
```

---

## Error Handling

### Python
```python
from cohere import CohereError, BadRequestError, TooManyRequestsError

try:
    response = co.chat(
        model="command-a-03-2025",
        messages=[{"role": "user", "content": "Hello"}]
    )
except BadRequestError as e:
    print(f"Invalid request: {e}")
except TooManyRequestsError as e:
    print(f"Rate limited: {e}")
    # Implement exponential backoff
except CohereError as e:
    print(f"Cohere error: {e}")
```

### TypeScript
```typescript
import { CohereError } from "cohere-ai";

try {
  const response = await cohere.chat({
    model: "command-a-03-2025",
    messages: [{ role: "user", content: "Hello" }]
  });
} catch (error) {
  if (error instanceof CohereError) {
    console.error(`Error: ${error.message}`);
    console.error(`Status: ${error.status}`);
  }
}
```

---

## Compatibility API (OpenAI SDK)

Use Cohere models with OpenAI SDK for easier migration.

### Python
```python
from openai import OpenAI

client = OpenAI(
    api_key="your-cohere-api-key",
    base_url="https://api.cohere.com/v1"
)

response = client.chat.completions.create(
    model="command-a-03-2025",
    messages=[
        {"role": "system", "content": "You are helpful."},
        {"role": "user", "content": "Hello!"}
    ]
)

print(response.choices[0].message.content)
```

### TypeScript
```typescript
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.COHERE_API_KEY,
  baseURL: "https://api.cohere.com/v1"
});

const response = await openai.chat.completions.create({
  model: "command-a-03-2025",
  messages: [
    { role: "system", content: "You are helpful." },
    { role: "user", content: "Hello!" }
  ]
});

console.log(response.choices[0].message.content);
```

---

## Production Best Practices

1. **Model Selection**: Use `command-r-plus` for quality, `command-r` for speed/cost
2. **Token Management**: Monitor token usage; implement token budgets
3. **Error Handling**: Catch and handle `TooManyRequestsError` with exponential backoff
4. **Streaming**: Use streaming for better real-time UX
5. **Tool Use**: Leverage tool calling for agentic workflows
6. **Embeddings**: Cache embeddings to avoid recomputation
7. **Fine-tuning**: Train custom models for domain-specific tasks
8. **Rate Limiting**: Implement request queuing for high-volume scenarios
9. **Monitoring**: Log API usage and model performance
10. **Security**: Keep API keys secure; use server-side implementations
11. **Fallback Strategy**: Have fallback models for redundancy
12. **Documentation**: Document your prompts and tool definitions

---

## Quick Reference

### Most Common Operations

```python
# Python: Basic chat
import cohere
co = cohere.ClientV2(api_key="key")
response = co.chat(
    model="command-a-03-2025",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.message.content[0].text)

# Python: Streaming
stream = co.chat_stream(model="command-a-03-2025", messages=[...])
for event in stream:
    if hasattr(event, 'delta'):
        print(event.delta.message.content[0].text, end="")

# Python: Embeddings
response = co.embed(
    model="embed-english-v3.0",
    input_type="search_document",
    texts=["text1", "text2"]
)

# Python: Rerank
response = co.rerank(
    model="rerank-english-v3.0",
    query="question",
    documents=["doc1", "doc2"]
)
```

```typescript
// TypeScript: Basic chat
import { CohereClientV2 } from "cohere-ai";
const cohere = new CohereClientV2();
const response = await cohere.chat({
  model: "command-a-03-2025",
  messages: [{ role: "user", content: "Hello" }]
});
console.log(response.message.content[0].text);

// TypeScript: Streaming
const stream = await cohere.chatStream({
  model: "command-a-03-2025",
  messages: [...]
});
for await (const event of stream) {
  if (event.type === "content-delta") {
    process.stdout.write(event.delta?.message?.content[0]?.text || "");
  }
}

// TypeScript: Embeddings
const embeddings = await cohere.embed({
  model: "embed-english-v3.0",
  inputType: "search_document",
  texts: ["text1", "text2"]
});

// TypeScript: Rerank
const rerank = await cohere.rerank({
  model: "rerank-english-v3.0",
  query: "question",
  documents: ["doc1", "doc2"]
});
```

### Response Structure
```json
{
  "message": {
    "role": "assistant",
    "content": [
      {
        "type": "text",
        "text": "response text"
      }
    ],
    "toolCalls": null
  },
  "usage": {
    "billed_units": 10,
    "input_tokens": 5,
    "output_tokens": 5
  }
}
```

### Environment Setup
```bash
export CO_API_KEY="your-api-key"
```

---

## Links & Resources

- **Official Docs**: https://docs.cohere.com
- **API Reference**: https://docs.cohere.com/reference/
- **Python SDK**: https://github.com/cohere-ai/cohere-python
- **TypeScript SDK**: https://github.com/cohere-ai/cohere-typescript
- **Dashboard**: https://dashboard.cohere.com
- **Cookbook**: https://github.com/cohere-ai/cohere-developer-experience
- **Community**: https://community.cohere.com

---

**Version**: SDK V2 | **Updated**: June 2026