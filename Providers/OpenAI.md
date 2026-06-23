# OpenAI SDK Reference

**Latest Update**: June 2026  
**Official Docs**: [platform.openai.com/docs](https://platform.openai.com/docs)  
**GitHub**: [openai/openai-python](https://github.com/openai/openai-python), [openai/openai-node](https://github.com/openai/openai-node)

## Table of Contents

- [Installation](#installation)
- [SDK Initialization](#sdk-initialization)
- [Authentication](#authentication)
- [Responses API (Core Generation)](#responses-api-core-generation)
- [Available Models](#available-models)
- [Multimodal (Vision)](#multimodal-vision)
- [File Uploads](#file-uploads)
- [Tool/Function Calling](#toolfunction-calling)
- [Structured Outputs (JSON Mode)](#structured-outputs-json-mode)
- [Embeddings](#embeddings)
- [Image Generation](#image-generation)
- [Fine-tuning](#fine-tuning)
- [Batch Processing](#batch-processing)
- [Error Handling](#error-handling)
- [Agents SDK](#agents-sdk)
- [Production Best Practices](#production-best-practices)
- [Quick Reference](#quick-reference)
- [Links & Resources](#links--resources)

---

## Installation

### Python
```bash
pip install openai
# Requires Python 3.7+
```

### TypeScript/JavaScript
```bash
npm install openai
# Requires TypeScript 4.9+ or Node.js 20+
```

---

## SDK Initialization

### Python
```python
from openai import OpenAI
import os

# Basic initialization
client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))

# Or use environment variable
# export OPENAI_API_KEY="your-key"
client = OpenAI()

# With custom configuration
client = OpenAI(
    api_key=os.environ.get("OPENAI_API_KEY"),
    timeout=30.0,
    max_retries=2
)
```

### Python - Async
```python
from openai import AsyncOpenAI

async_client = AsyncOpenAI(api_key="your-key")
# Use with await for async operations
```

### Python - Azure OpenAI
```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key=os.environ.get("AZURE_OPENAI_KEY"),
    api_version="2024-10-01-preview",
    azure_endpoint=os.environ.get("AZURE_OPENAI_ENDPOINT")
)
```

### TypeScript/JavaScript
```typescript
import OpenAI from "openai";

// Basic initialization
const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Or use environment variable
// export OPENAI_API_KEY="your-key"
const client = new OpenAI();

// With custom configuration
const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
  timeout: 30000,  // milliseconds
  maxRetries: 2
});
```

### TypeScript - Azure OpenAI
```typescript
import { AzureOpenAI } from "openai";

const client = new AzureOpenAI({
  apiKey: process.env.AZURE_OPENAI_KEY,
  apiVersion: "2024-10-01-preview",
  baseURL: `${process.env.AZURE_OPENAI_ENDPOINT}/openai/deployments/${process.env.AZURE_DEPLOYMENT_NAME}`
});
```

---

## Authentication

OpenAI uses **API key authentication** via the `Authorization: Bearer` header.

```bash
# Set environment variable
export OPENAI_API_KEY="your-api-key"
```

Get your API key from [OpenAI Platform](https://platform.openai.com/account/api-keys).

---

## Responses API (Core Generation)

The primary API for text generation with OpenAI models.

### Python - Unary (Simple)
```python
from openai import OpenAI

client = OpenAI()

response = client.responses.create(
    model="gpt-5.5",
    input="Write a one-sentence bedtime story about a unicorn"
)

print(response.output_text)
```

### Python - Chat API (Legacy but Still Popular)
```python
response = client.chat.completions.create(
    model="gpt-5.5",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain machine learning"}
    ],
    temperature=0.7,
    max_tokens=1024
)

print(response.choices[0].message.content)
```

### Python - Streaming
```python
stream = client.chat.completions.create(
    model="gpt-5.5",
    messages=[
        {"role": "user", "content": "Write a short story about AI"}
    ],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### Python - Conversation (Multi-turn)
```python
messages = []

# First turn
messages.append({"role": "user", "content": "What is machine learning?"})

response = client.chat.completions.create(
    model="gpt-5.5",
    messages=messages
)

assistant_message = response.choices[0].message.content
messages.append({"role": "assistant", "content": assistant_message})

print(f"User: What is machine learning?")
print(f"Assistant: {assistant_message}\n")

# Second turn
messages.append({"role": "user", "content": "Tell me about neural networks"})

response = client.chat.completions.create(
    model="gpt-5.5",
    messages=messages
)

print(f"Assistant: {response.choices[0].message.content}")
```

### TypeScript - Unary
```typescript
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

const response = await client.responses.create({
  model: "gpt-5.5",
  input: "Write a one-sentence bedtime story about a unicorn"
});

console.log(response.outputText);
```

### TypeScript - Chat API
```typescript
const response = await client.chat.completions.create({
  model: "gpt-5.5",
  messages: [
    { role: "system", content: "You are a helpful assistant." },
    { role: "user", content: "Explain machine learning" }
  ],
  temperature: 0.7,
  maxTokens: 1024
});

console.log(response.choices[0].message.content);
```

### TypeScript - Streaming
```typescript
const stream = client.chat.completions.stream({
  model: "gpt-5.5",
  messages: [
    { role: "user", content: "Write a short story about AI" }
  ]
});

stream.on("text", (text) => {
  process.stdout.write(text);
});

const finalMessage = await stream.finalMessage();
console.log("Done!");
```

### Common Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `model` | string | Model ID (see below) |
| `input` | string | Text prompt (Responses API) |
| `messages` | array | Message history (Chat API) |
| `temperature` | float | 0.0-2.0; higher = more creative |
| `max_tokens` | int | Maximum response length |
| `top_p` | float | Nucleus sampling (0-1) |
| `frequency_penalty` | float | -2.0 to 2.0 |
| `presence_penalty` | float | -2.0 to 2.0 |
| `seed` | int | Reproducibility |
| `stream` | bool | Enable streaming |

---

## Available Models

| Model | Context | Cost | Best For |
|-------|---------|------|----------|
| `gpt-5.5` | Large | Medium | General-purpose |
| `gpt-5.4` | Large | High | Complex reasoning |
| `gpt-4o` | 128K | Low | Balanced performance |
| `gpt-4-turbo` | 128K | Medium | Coding, analysis |
| `gpt-3.5-turbo` | 4K | Very Low | Simple tasks |

**Tip**: Use latest stable model; check [platform.openai.com/docs/models](https://platform.openai.com/docs/models) for current availability.

---

## Multimodal (Vision)

### Python - Image Input
```python
from openai import OpenAI
import base64

client = OpenAI()

# From file (base64 encoded)
with open("image.jpg", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Describe this image"},
                {
                    "type": "image_url",
                    "image_url": {
                        "url": f"data:image/jpeg;base64,{image_data}"
                    }
                }
            ]
        }
    ],
    max_tokens=1024
)

print(response.choices[0].message.content)
```

### Python - URL-based Image
```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What's in this image?"},
                {
                    "type": "image_url",
                    "image_url": {
                        "url": "https://example.com/image.jpg"
                    }
                }
            ]
        }
    ]
)
```

### TypeScript - Image Input
```typescript
import fs from "fs";
import OpenAI from "openai";

const client = new OpenAI();

const imageBuffer = fs.readFileSync("image.jpg");
const base64Image = imageBuffer.toString("base64");

const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Describe this image" },
        {
          type: "image_url",
          image_url: {
            url: `data:image/jpeg;base64,${base64Image}`
          }
        }
      ]
    }
  ],
  maxTokens: 1024
});

console.log(response.choices[0].message.content);
```

---

## File Uploads

Upload files for processing (PDFs, documents, etc.).

### Python
```python
# Upload file
with open("document.pdf", "rb") as f:
    response = client.files.create(
        file=f,
        purpose="assistants"  # or "batch", "fine-tune"
    )
    file_id = response.id

print(f"File uploaded: {file_id}")

# List files
files = client.files.list()
for file in files.data:
    print(f"File: {file.id} ({file.filename})")

# Delete file
client.files.delete(file_id)
```

### TypeScript
```typescript
import fs from "fs";

const file = fs.createReadStream("document.pdf");

const response = await client.files.create({
  file: file,
  purpose: "assistants"
});

const fileId = response.id;
console.log(`File uploaded: ${fileId}`);

// Delete file
await client.files.delete(fileId);
```

---

## Tool/Function Calling

Enable GPT to call external functions.

### Python
```python
import json

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_current_temperature",
            "description": "Get the current temperature for a given location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "The city and state, e.g. San Francisco, CA"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "description": "Temperature unit"
                    }
                },
                "required": ["location"]
            }
        }
    }
]

messages = [
    {"role": "user", "content": "What's the weather in San Francisco?"}
]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    tools=tools,
    tool_choice="auto"
)

# Check for tool calls
if response.choices[0].message.tool_calls:
    for tool_call in response.choices[0].message.tool_calls:
        print(f"Function: {tool_call.function.name}")
        print(f"Arguments: {tool_call.function.arguments}")
        
        # Execute function
        arguments = json.loads(tool_call.function.arguments)
        
        if tool_call.function.name == "get_current_temperature":
            # Your function implementation
            result = "72 degrees fahrenheit"
        else:
            result = "Function not found"
        
        # Continue conversation
        messages.append(response.choices[0].message)
        messages.append({
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": result
        })
        
        final_response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=tools
        )
        
        print(f"Final: {final_response.choices[0].message.content}")
```

### TypeScript
```typescript
const tools: OpenAI.Chat.ChatCompletionTool[] = [
  {
    type: "function",
    function: {
      name: "get_current_temperature",
      description: "Get the current temperature for a given location",
      parameters: {
        type: "object",
        properties: {
          location: {
            type: "string",
            description: "The city and state"
          },
          unit: {
            type: "string",
            enum: ["celsius", "fahrenheit"]
          }
        },
        required: ["location"]
      }
    }
  }
];

const messages: OpenAI.Chat.ChatCompletionMessageParam[] = [
  { role: "user", content: "What's the weather in San Francisco?" }
];

let response = await client.chat.completions.create({
  model: "gpt-4o",
  messages,
  tools,
  toolChoice: "auto"
});

const toolCalls = response.choices[0].message.toolCalls;
if (toolCalls) {
  for (const toolCall of toolCalls) {
    console.log(`Function: ${toolCall.function.name}`);
    console.log(`Arguments: ${toolCall.function.arguments}`);
    
    const args = JSON.parse(toolCall.function.arguments);
    let result = "72 degrees fahrenheit";
    
    messages.push(response.choices[0].message);
    messages.push({
      role: "tool",
      tool_call_id: toolCall.id,
      content: result
    });
    
    const finalResponse = await client.chat.completions.create({
      model: "gpt-4o",
      messages,
      tools
    });
    
    console.log(`Final: ${finalResponse.choices[0].message.content}`);
  }
}
```

---

## Structured Outputs (JSON Mode)

Force JSON-formatted responses.

### Python
```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "user",
            "content": "Extract name and age from: John Smith, 28 years old"
        }
    ],
    response_format={"type": "json_object"}
)

import json
data = json.loads(response.choices[0].message.content)
print(f"Name: {data['name']}, Age: {data['age']}")
```

### TypeScript
```typescript
const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [
    {
      role: "user",
      content: "Extract name and age from: John Smith, 28 years old"
    }
  ],
  responseFormat: { type: "json_object" }
});

const data = JSON.parse(response.choices[0].message.content || "{}");
console.log(`Name: ${data.name}, Age: ${data.age}`);
```

---

## Embeddings

Generate vector embeddings for semantic search.

### Python
```python
response = client.embeddings.create(
    model="text-embedding-3-small",
    input=["This is a sample sentence", "Another sentence here"]
)

for i, embedding in enumerate(response.data):
    print(f"Text {i}: {len(embedding.embedding)} dimensions")
    print(f"First 5 values: {embedding.embedding[:5]}")
```

### TypeScript
```typescript
const response = await client.embeddings.create({
  model: "text-embedding-3-small",
  input: ["This is a sample sentence", "Another sentence here"]
});

response.data.forEach((emb, i) => {
  console.log(`Text ${i}: ${emb.embedding.length} dimensions`);
  console.log(`First 5 values: ${emb.embedding.slice(0, 5)}`);
});
```

### Embedding Models

| Model | Dimension | Use Case |
|-------|-----------|----------|
| `text-embedding-3-large` | 3072 | Highest quality embeddings |
| `text-embedding-3-small` | 1536 | Fast, cost-efficient |

---

## Image Generation

Generate images from text prompts.

### Python
```python
response = client.images.generate(
    model="dall-e-3",
    prompt="A serene mountain landscape at sunset",
    n=1,
    size="1024x1024",
    quality="hd",
    style="natural"
)

# Get image URL or B64
image_url = response.data[0].url
print(f"Image URL: {image_url}")

# Or save B64
# with open("image.png", "wb") as f:
#     f.write(base64.b64decode(response.data[0].b64_json))
```

### TypeScript
```typescript
const response = await client.images.generate({
  model: "dall-e-3",
  prompt: "A serene mountain landscape at sunset",
  n: 1,
  size: "1024x1024",
  quality: "hd",
  style: "natural"
});

const imageUrl = response.data[0].url;
console.log(`Image URL: ${imageUrl}`);
```

---

## Fine-tuning

Create custom models trained on your data.

### Python
```python
import time

# Upload training file (JSONL format)
with open("training_data.jsonl", "rb") as f:
    file_response = client.files.create(
        file=f,
        purpose="fine-tune"
    )

file_id = file_response.id

# Create fine-tuning job
job = client.fine_tuning.jobs.create(
    model="gpt-4o-mini",
    training_file=file_id,
    hyperparameters={
        "learning_rate_multiplier": 1.0,
        "batch_size": 4,
        "n_epochs": 3
    }
)

print(f"Job ID: {job.id}")
print(f"Status: {job.status}")

# Poll for completion
while job.status not in ["succeeded", "failed"]:
    time.sleep(10)
    job = client.fine_tuning.jobs.retrieve(job.id)
    print(f"Status: {job.status}")

if job.status == "succeeded":
    print(f"Fine-tuned model: {job.fine_tuned_model}")
    
    # Use fine-tuned model
    response = client.chat.completions.create(
        model=job.fine_tuned_model,
        messages=[{"role": "user", "content": "Hello"}]
    )
```

### TypeScript
```typescript
// Upload training file
const file = fs.createReadStream("training_data.jsonl");
const fileResponse = await client.files.create({
  file: file,
  purpose: "fine-tune"
});

const fileId = fileResponse.id;

// Create job
const job = await client.fineTuning.jobs.create({
  model: "gpt-4o-mini",
  training_file: fileId,
  hyperparameters: {
    learning_rate_multiplier: 1.0,
    batch_size: 4,
    n_epochs: 3
  }
});

console.log(`Job ID: ${job.id}`);

// Poll for completion
let completedJob = job;
while (!["succeeded", "failed"].includes(completedJob.status)) {
  await new Promise(resolve => setTimeout(resolve, 10000));
  completedJob = await client.fineTuning.jobs.retrieve(job.id);
  console.log(`Status: ${completedJob.status}`);
}
```

### Training Data Format (JSONL)
```jsonl
{"messages": [{"role": "user", "content": "...", "role": "assistant", "content": "..."}]}
{"messages": [{"role": "user", "content": "...", "role": "assistant", "content": "..."}]}
```

---

## Batch Processing

Process many requests efficiently at lower cost.

### Python
```python
import json

# Create batch
batch_input_file_id = client.files.create(
    file=open("batch_input.jsonl", "rb"),
    purpose="batch"
).id

batch = client.batches.create(
    input_file_id=batch_input_file_id,
    endpoint="/v1/chat/completions",
    completion_window="24h"
)

print(f"Batch ID: {batch.id}")

# Poll for completion
import time
while batch.status not in ["completed", "failed"]:
    time.sleep(10)
    batch = client.batches.retrieve(batch.id)

# Get results
if batch.status == "completed":
    results = client.files.content(batch.output_file_id).content
    print(results)
```

---

## Error Handling

### Python
```python
from openai import RateLimitError, APIConnectionError, APIError

try:
    response = client.chat.completions.create(
        model="gpt-5.5",
        messages=[{"role": "user", "content": "Hello"}]
    )
except RateLimitError as e:
    print(f"Rate limited: {e}")
except APIConnectionError as e:
    print(f"Connection error: {e}")
except APIError as e:
    print(f"API error: {e}")
```

### TypeScript
```typescript
try {
  const response = await client.chat.completions.create({
    model: "gpt-5.5",
    messages: [{ role: "user", content: "Hello" }]
  });
} catch (error) {
  if (error instanceof OpenAI.RateLimitError) {
    console.error("Rate limited");
  } else if (error instanceof OpenAI.APIConnectionError) {
    console.error("Connection error");
  } else if (error instanceof OpenAI.APIError) {
    console.error(`API error: ${error.message}`);
  }
}
```

---

## Agents SDK

Build autonomous agents with tool use and function calling.

### TypeScript
```typescript
import { Agent, run } from "@openai/agents";

const agent = new Agent({
  name: "Data Analyst",
  instructions: "You are a helpful data analyst assistant",
  tools: [
    // Your tools here
  ]
});

const result = await run(agent, "Analyze the data in file.csv");
console.log(result.finalOutput);
```

---

## Production Best Practices

1. **API Key Security**: Use environment variables; never hardcode keys
2. **Error Handling**: Catch rate limits, timeouts, and API errors
3. **Retries**: Implement exponential backoff for transient failures
4. **Streaming**: Use streaming for long responses to improve UX
5. **Token Management**: Monitor token usage for cost control
6. **Rate Limiting**: Implement request queuing and backoff
7. **Model Selection**: Use latest stable model; pin versions in production
8. **Timeouts**: Set reasonable timeouts (default 10 min)
9. **Monitoring**: Log all API calls and errors
10. **Cost Optimization**: Use batch API for non-urgent requests
11. **Tool Design**: Keep tool schemas simple and well-documented
12. **Testing**: Test error scenarios and edge cases

---

## Quick Reference

### Most Common Operations

```python
# Python: Chat completion
from openai import OpenAI
client = OpenAI()
response = client.chat.completions.create(
    model="gpt-5.5",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)

# Python: Streaming
stream = client.chat.completions.create(
    model="gpt-5.5",
    messages=[...],
    stream=True
)
for chunk in stream:
    print(chunk.choices[0].delta.content or "", end="")

# Python: Function calling
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    tools=tools
)

# Python: Embeddings
response = client.embeddings.create(
    model="text-embedding-3-small",
    input=["text1", "text2"]
)
```

```typescript
// TypeScript: Chat completion
import OpenAI from "openai";
const client = new OpenAI();
const response = await client.chat.completions.create({
  model: "gpt-5.5",
  messages: [{ role: "user", content: "Hello" }]
});

// TypeScript: Streaming
const stream = client.chat.completions.stream({
  model: "gpt-5.5",
  messages: [...]
});
stream.on("text", (text) => process.stdout.write(text));

// TypeScript: Function calling
const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [...],
  tools
});

// TypeScript: Embeddings
const response = await client.embeddings.create({
  model: "text-embedding-3-small",
  input: ["text1", "text2"]
});
```

### Response Structure
```json
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1234567890,
  "model": "gpt-5.5",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "response text"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 20,
    "total_tokens": 30
  }
}
```

### Environment Setup
```bash
export OPENAI_API_KEY="your-api-key"
```

---

## Links & Resources

- **Official Docs**: https://platform.openai.com/docs
- **API Reference**: https://platform.openai.com/docs/api-reference
- **Python SDK**: https://github.com/openai/openai-python
- **TypeScript SDK**: https://github.com/openai/openai-node
- **Platform**: https://platform.openai.com
- **Cookbook**: https://github.com/openai/openai-cookbook
- **Models**: https://platform.openai.com/docs/models
- **Community**: https://community.openai.com

---

**Version**: SDK Latest | **Updated**: June 2026