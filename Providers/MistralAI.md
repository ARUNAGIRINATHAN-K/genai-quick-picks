# Mistral AI SDK Reference

**Latest Update**: June 2026  
**Official Docs**: [docs.mistral.ai](https://docs.mistral.ai)  
**GitHub**: [mistralai/client-python](https://github.com/mistralai/client-python), [mistralai/client-ts](https://github.com/mistralai/client-ts)

## Installation

### Python
```bash
pip install mistralai
# Requires Python 3.9+
```\

### TypeScript/JavaScript
```bash
npm install @mistralai/mistralai
# ESM-only, Node.js 18+
```

---

## SDK Initialization

### Python
```python
import os
from mistralai import Mistral

# Basic initialization
client = Mistral(api_key=os.environ["MISTRAL_API_KEY"])

# Custom server URL (e.g., for Azure)
client = Mistral(
    api_key=os.environ["MISTRAL_API_KEY"],
    server_url="https://api.mistral.ai"
)

# With custom HTTP client configuration
from httpx import Client as HttpClient
http_client = HttpClient(timeout=30.0)
client = Mistral(
    api_key=os.environ["MISTRAL_API_KEY"],
    http_client=http_client
)
```

### TypeScript/JavaScript
```typescript
import { Mistral } from "@mistralai/mistralai";

// Basic initialization
const mistral = new Mistral({
  apiKey: process.env.MISTRAL_API_KEY
});

// Custom server URL
const mistral = new Mistral({
  apiKey: process.env.MISTRAL_API_KEY,
  serverURL: "https://api.mistral.ai"
});

// Custom fetch options
const mistral = new Mistral({
  apiKey: process.env.MISTRAL_API_KEY,
  fetchOptions: {
    timeout: 30000
  }
});
```

---

## Authentication

Mistral uses **API key authentication** via the `Authorization: Bearer` header.

```bash
# Set environment variable
export MISTRAL_API_KEY="your-api-key"
```

Get your API key from [Mistral Studio](https://studio.mistral.ai) (free tier available).

---

## Chat Completions (Core Generation)

### Python - Unary (Simple)
```python
response = client.chat.complete(
    model="mistral-large-latest",
    messages=[
        {"role": "user", "content": "What is Mistral AI?"}
    ]
)

print(response.choices[0].message.content)
```

### Python - Streaming
```python
response = client.chat.stream(
    model="mistral-large-latest",
    messages=[
        {"role": "user", "content": "Explain machine learning"}
    ]
)

for chunk in response:
    if chunk.data.choices[0].delta.content:
        print(chunk.data.choices[0].delta.content, end="")
```

### TypeScript - Unary
```typescript
const response = await mistral.chat.complete({
  model: "mistral-large-latest",
  messages: [
    { role: "user", content: "What is Mistral AI?" }
  ]
});

console.log(response.choices[0].message.content);
```

### TypeScript - Streaming
```typescript
const response = await mistral.chat.stream({
  model: "mistral-large-latest",
  messages: [
    { role: "user", content: "Explain machine learning" }
  ]
});

for await (const chunk of response) {
  if (chunk.data.choices[0].delta.content) {
    process.stdout.write(chunk.data.choices[0].delta.content);
  }
}
```

### Common Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `model` | string | Model ID (e.g., `mistral-large-latest`) |
| `messages` | array | Message history with `role` and `content` |
| `temperature` | float | 0.0-1.0; higher = more creative (default: 0.3) |
| `max_tokens` | int | Maximum tokens in response |
| `top_p` | float | Nucleus sampling (0-1) |
| `top_k` | int | Top-K sampling |
| `min_tokens` | int | Minimum tokens to generate |
| `random_seed` | int | For reproducibility |

---

## Available Models

| Model | Context | Use Case |
|-------|---------|----------|
| `mistral-large-latest` | 128K | General-purpose, reasoning |
| `mistral-medium-latest` | 32K | Balanced speed/quality |
| `mistral-small-latest` | 32K | Fast, cost-efficient |
| `codestral-latest` | 32K | Code generation/analysis |
| `pixtral-12b-latest` | 128K | Vision/multimodal |

**Note**: Always pin model versions in production (e.g., `mistral-large-2512`) rather than using `-latest`.

---

## Multimodal (Vision)

### Python - Image Input
```python
import base64

# From file
with open("image.jpg", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.chat.complete(
    model="pixtral-12b-latest",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Describe this image"},
                {
                    "type": "image",
                    "image_url": f"data:image/jpeg;base64,{image_data}"
                }
            ]
        }
    ]
)

print(response.choices[0].message.content)
```

### TypeScript - Image Input
```typescript
import fs from "fs";
import path from "path";

const imageBuffer = fs.readFileSync("image.jpg");
const base64Image = imageBuffer.toString("base64");

const response = await mistral.chat.complete({
  model: "pixtral-12b-latest",
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Describe this image" },
        {
          type: "image",
          imageUrl: `data:image/jpeg;base64,${base64Image}`
        }
      ]
    }
  ]
});

console.log(response.choices[0].message.content);
```

---

## Tool/Function Calling

Mistral automatically detects when to call tools based on user requests.

### Python
```python
response = client.chat.complete(
    model="mistral-large-latest",
    messages=[
        {
            "role": "user",
            "content": "What's the weather in Paris?"
        }
    ],
    tools=[
        {
            "type": "function",
            "function": {
                "name": "get_weather",
                "description": "Get the current weather",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {
                            "type": "string",
                            "description": "City name"
                        },
                        "unit": {
                            "type": "string",
                            "enum": ["celsius", "fahrenheit"]
                        }
                    },
                    "required": ["location"]
                }
            }
        }
    ]
)

# Check if tool was called
choice = response.choices[0]
if choice.message.tool_calls:
    for tool_call in choice.message.tool_calls:
        print(f"Tool: {tool_call.function.name}")
        print(f"Args: {tool_call.function.arguments}")
        
        # Execute tool and send result back
        if tool_call.function.name == "get_weather":
            # Your implementation
            weather_result = "Sunny, 20°C"
            
            # Continue conversation with tool result
            messages = [
                {"role": "user", "content": "What's the weather in Paris?"},
                {"role": "assistant", "content": "", "tool_calls": [tool_call]},
                {
                    "role": "tool",
                    "content": weather_result,
                    "tool_call_id": tool_call.id
                }
            ]
            final_response = client.chat.complete(
                model="mistral-large-latest",
                messages=messages
            )
            print(final_response.choices[0].message.content)
```

### TypeScript
```typescript
const response = await mistral.chat.complete({
  model: "mistral-large-latest",
  messages: [
    {
      role: "user",
      content: "What's the weather in Paris?"
    }
  ],
  tools: [
    {
      type: "function",
      function: {
        name: "get_weather",
        description: "Get the current weather",
        parameters: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "City name"
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
  ]
});

const choice = response.choices[0];
if (choice.message.toolCalls) {
  for (const toolCall of choice.message.toolCalls) {
    console.log(`Tool: ${toolCall.function.name}`);
    console.log(`Args: ${toolCall.function.arguments}`);
    
    // Execute and continue
    const weatherResult = "Sunny, 20°C";
    const messages = [
      { role: "user" as const, content: "What's the weather in Paris?" },
      { role: "assistant" as const, content: "", toolCalls: [toolCall] },
      {
        role: "tool" as const,
        content: weatherResult,
        toolCallId: toolCall.id
      }
    ];
    
    const finalResponse = await mistral.chat.complete({
      model: "mistral-large-latest",
      messages
    });
    console.log(finalResponse.choices[0].message.content);
  }
}
```

---

## Structured Outputs (JSON Mode)

Force model to respond with valid JSON schema.

### Python
```python
response = client.chat.complete(
    model="mistral-large-latest",
    messages=[
        {
            "role": "user",
            "content": "Extract the person's name and age"
        }
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "person_info",
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "age": {"type": "integer"}
                },
                "required": ["name", "age"]
            }
        }
    }
)

# Parse JSON response
import json
data = json.loads(response.choices[0].message.content)
print(f"Name: {data['name']}, Age: {data['age']}")
```

### TypeScript
```typescript
const response = await mistral.chat.complete({
  model: "mistral-large-latest",
  messages: [
    {
      role: "user",
      content: "Extract the person's name and age"
    }
  ],
  responseFormat: {
    type: "json_schema",
    jsonSchema: {
      name: "person_info",
      schema: {
        type: "object",
        properties: {
          name: { type: "string" },
          age: { type: "integer" }
        },
        required: ["name", "age"]
      }
    }
  }
});

const data = JSON.parse(response.choices[0].message.content);
console.log(`Name: ${data.name}, Age: ${data.age}`);
```

---

## Embeddings

Generate vector embeddings for semantic search and RAG.

### Python
```python
response = client.embeddings.create(
    model="mistral-embed",
    inputs=[
        "Embed this sentence.",
        "As well as this one.",
    ]
)

for embedding in response.data:
    print(f"Embedding dimension: {len(embedding.embedding)}")
    print(f"First 5 values: {embedding.embedding[:5]}")
```

### TypeScript
```typescript
const response = await mistral.embeddings.create({
  model: "mistral-embed",
  input: [
    "Embed this sentence.",
    "As well as this one."
  ]
});

for (const embedding of response.data) {
  console.log(`Embedding dimension: ${embedding.embedding.length}`);
  console.log(`First 5 values: ${embedding.embedding.slice(0, 5)}`);
}
```

### Embedding Models

| Model | Dimension | Use Case |
|-------|-----------|----------|
| `mistral-embed` | 1024 | General-purpose embeddings |

---

## Async Support

### Python
```python
import asyncio
from mistralai import Mistral

async def main():
    async with Mistral(api_key="your-key") as client:
        response = await client.chat.complete_async(
            model="mistral-large-latest",
            messages=[
                {"role": "user", "content": "Hello!"}
            ]
        )
        print(response.choices[0].message.content)

asyncio.run(main())
```

### TypeScript
```typescript
const response = await mistral.chat.complete({
  model: "mistral-large-latest",
  messages: [
    { role: "user", content: "Hello!" }
  ]
});

console.log(response.choices[0].message.content);
// All methods are async by default
```

---

## Error Handling

### Python
```python
from mistralai.exceptions import (
    MistralError,
    HTTPValidationError,
    ResponseValidationError
)

try:
    response = client.chat.complete(
        model="mistral-large-latest",
        messages=[{"role": "user", "content": "Hello"}]
    )
except HTTPValidationError as e:
    print(f"Validation error: {e.status_code}")
except ResponseValidationError as e:
    print(f"Response parsing error: {e.cause}")
except MistralError as e:
    print(f"API error: {str(e)}")
```

### TypeScript
```typescript
import {
  MistralError,
  InvalidRequestError,
  RequestTimeoutError,
  ResponseValidationError
} from "@mistralai/mistralai";

try {
  const response = await mistral.chat.complete({
    model: "mistral-large-latest",
    messages: [{ role: "user", content: "Hello" }]
  });
} catch (e) {
  if (e instanceof RequestTimeoutError) {
    console.error("Request timed out");
  } else if (e instanceof InvalidRequestError) {
    console.error("Invalid request:", e.message);
  } else if (e instanceof ResponseValidationError) {
    console.error("Response validation failed:", e.pretty());
  } else if (e instanceof MistralError) {
    console.error("Mistral API error:", e.message);
  }
}
```

---

## Streaming Response Objects

### Python Stream Event
```python
# Each stream chunk is a StreamEvent object
chunk = {
    "id": "string",
    "object": "text_completion.chunk",
    "created": 1234567890,
    "model": "mistral-large-latest",
    "data": {
        "choices": [
            {
                "index": 0,
                "delta": {"content": "generated text"},
                "finish_reason": None  # or "stop", "length", etc.
            }
        ]
    }
}
```

### TypeScript Stream Chunk
```typescript
interface StreamChunk {
  id: string;
  object: string;
  created: number;
  model: string;
  data: {
    choices: Array<{
      index: number;
      delta: { content?: string };
      finishReason?: string;
    }>;
  };
}
```

---

## Models List & Management

### Python
```python
# List available models
models = client.models.list()
for model in models.data:
    print(f"{model.id} - {model.created}")

# Get specific model
model = client.models.get("mistral-large-latest")
print(model)

# Delete fine-tuned model
deleted = client.models.delete("ft:mistral-7b:custom-id")
print(deleted.deleted)
```

### TypeScript
```typescript
// List available models
const models = await mistral.models.list();
for (const model of models.data) {
  console.log(`${model.id} - ${model.created}`);
}

// Get specific model
const model = await mistral.models.get("mistral-large-latest");
console.log(model);

// Delete fine-tuned model
const deleted = await mistral.models.delete("ft:mistral-7b:custom-id");
console.log(deleted.deleted);
```

---

## Advanced: File Uploads & Documents

### Python - Upload Document
```python
# Upload file for RAG/document processing
with open("document.pdf", "rb") as f:
    uploaded_file = client.files.upload(file=f)
    print(f"File ID: {uploaded_file.id}")

# Use in chat
response = client.chat.complete(
    model="mistral-large-latest",
    messages=[
        {
            "role": "user",
            "content": "Summarize the uploaded document"
        }
    ],
    # File context can be passed as needed
)
```

### TypeScript - Upload Document
```typescript
import fs from "fs";

// Upload file
const fileStream = fs.createReadStream("document.pdf");
const uploadedFile = await mistral.files.upload({
  file: fileStream
});
console.log(`File ID: ${uploadedFile.id}`);

// Use in chat
const response = await mistral.chat.complete({
  model: "mistral-large-latest",
  messages: [
    {
      role: "user",
      content: "Summarize the uploaded document"
    }
  ]
});
```

---

## Production Best Practices

1. **Pin Model Versions**: Use specific versions (`mistral-large-2512`) rather than `-latest` aliases
2. **Set Timeouts**: Configure HTTP timeouts to prevent hanging requests
3. **Retry Logic**: Implement exponential backoff for transient errors
4. **Rate Limiting**: Monitor quota usage and implement request batching
5. **API Key Security**: Never embed keys in client-side code; use server-side proxies
6. **Response Validation**: Always validate streaming responses for `finish_reason`
7. **Token Budgets**: Monitor `max_tokens` to control costs
8. **Caching**: Cache embeddings and frequently-requested completions
9. **Monitoring**: Log model usage, latency, and error rates
10. **Error Recovery**: Implement circuit breakers for cascading failures

---

## Quick Reference

### Most Common Operations

```python
# Python: Basic chat
from mistralai import Mistral
client = Mistral(api_key="key")
response = client.chat.complete(
    model="mistral-large-latest",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)

# Python: Streaming
for chunk in client.chat.stream(model="mistral-large-latest", messages=[...]):
    print(chunk.data.choices[0].delta.content, end="")

# Python: Embeddings
embeddings = client.embeddings.create(model="mistral-embed", inputs=["text"])
print(embeddings.data[0].embedding)
```

```typescript
// TypeScript: Basic chat
import { Mistral } from "@mistralai/mistralai";
const mistral = new Mistral({ apiKey: "key" });
const response = await mistral.chat.complete({
  model: "mistral-large-latest",
  messages: [{ role: "user", content: "Hello" }]
});
console.log(response.choices[0].message.content);

// TypeScript: Streaming
const stream = await mistral.chat.stream({
  model: "mistral-large-latest",
  messages: [...]
});
for await (const chunk of stream) {
  process.stdout.write(chunk.data.choices[0].delta.content || "");
}

// TypeScript: Embeddings
const embeddings = await mistral.embeddings.create({
  model: "mistral-embed",
  input: ["text"]
});
console.log(embeddings.data[0].embedding);
```

### Response Structure
```python
# Chat completion response
{
    "id": "unique-id",
    "object": "chat.completion",
    "created": 1234567890,
    "model": "mistral-large-latest",
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": "response text",
                "tool_calls": []  # if tools were called
            },
            "finish_reason": "stop"  # "stop", "length", "tool_calls"
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
# Set API key
export MISTRAL_API_KEY="your-key-here"

# Optional: Set custom server
export MISTRAL_SERVER_URL="https://api.mistral.ai"
```

---

## Links & Resources

- **Official Docs**: https://docs.mistral.ai
- **API Reference**: https://docs.mistral.ai/api/
- **Python SDK GitHub**: https://github.com/mistralai/client-python
- **TypeScript SDK GitHub**: https://github.com/mistralai/client-ts
- **Mistral Studio**: https://studio.mistral.ai (playground)
- **Changelog**: https://docs.mistral.ai/changelog/
- **Discord Community**: https://discord.com/invite/mistralai

---

**Version**: SDK v0.3+ | **Updated**: June 2026
