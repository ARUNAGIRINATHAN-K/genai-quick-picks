# Anthropic (Claude) SDK Reference

**Latest Update**: June 2026  
**Official Docs**: [docs.anthropic.com](https://docs.anthropic.com)  
**GitHub**: [anthropics/anthropic-sdk-python](https://github.com/anthropics/anthropic-sdk-python), [anthropics/anthropic-sdk-typescript](https://github.com/anthropics/anthropic-sdk-typescript)

## Table of Contents

- [Installation](#installation)
- [SDK Initialization](#sdk-initialization)
- [Authentication](#authentication)
- [Messages API (Core Generation)](#messages-api-core-generation)
- [Available Models](#available-models)
- [Multimodal (Vision)](#multimodal-vision)
- [Tool Use (Function Calling)](#tool-use-function-calling)
- [Structured Outputs](#structured-outputs)
- [Prompt Caching](#prompt-caching)
- [Batch API](#batch-api)
- [Error Handling](#error-handling)
- [Token Counting](#token-counting)
- [Usage & Token Management](#usage--token-management)
- [Async Operations](#async-operations)
- [Production Best Practices](#production-best-practices)
- [Quick Reference](#quick-reference)
- [Links & Resources](#links--resources)

---

## Installation

### Python
```bash
pip install anthropic
# Requires Python 3.7+
```

### TypeScript/JavaScript
```bash
npm install @anthropic-ai/sdk
# Requires TypeScript 4.9+ or Node.js 20+
```

---

## SDK Initialization

### Python
```python
import anthropic
import os

# Basic initialization
client = anthropic.Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))

# Or use environment variable
# export ANTHROPIC_API_KEY="your-key"
client = anthropic.Anthropic()

# With custom configuration
client = anthropic.Anthropic(
    api_key=os.environ.get("ANTHROPIC_API_KEY"),
    timeout=30.0,
    max_retries=3
)
```

### Python - Async
```python
import anthropic

async_client = anthropic.AsyncAnthropic(api_key="your-key")
# Use with await for async operations
```

### Python - Cloud Deployments
```python
# AWS Bedrock
client = anthropic.AnthropicBedrock(
    aws_region="us-east-1",
    aws_access_key="...",
    aws_secret_key="..."
)

# Google Vertex AI
client = anthropic.AnthropicVertex(
    project_id="your-project",
    region="us-central1"
)

# Azure
client = anthropic.AnthropicAzure(
    api_key="...",
    base_url="https://<your-resource>.openai.azure.com/"
)
```

### TypeScript/JavaScript
```typescript
import Anthropic from "@anthropic-ai/sdk";

// Basic initialization
const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Or use environment variable
// export ANTHROPIC_API_KEY="your-key"
const client = new Anthropic();

// With custom configuration
const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  timeout: 30000,  // milliseconds
  maxRetries: 3
});
```

### TypeScript - Cloud Deployments
```typescript
// AWS Bedrock
import { AnthropicBedrock } from "@anthropic-ai/bedrock-sdk";
const client = new AnthropicBedrock({
  awsRegion: "us-east-1"
});

// Google Vertex AI
import { AnthropicVertex } from "@anthropic-ai/vertex-sdk";
const client = new AnthropicVertex({
  projectId: "your-project",
  region: "us-central1"
});
```

---

## Authentication

Anthropic uses **API key authentication** via the `x-api-key` header.

```bash
# Set environment variable
export ANTHROPIC_API_KEY="your-api-key"
```

Get your API key from [Anthropic Console](https://console.anthropic.com/account/keys).

---

## Messages API (Core Generation)

### Python - Unary (Simple)
```python
import anthropic

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Hello, Claude"}
    ]
)

print(message.content[0].text)
```

### Python - Streaming
```python
with client.messages.stream(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Write a poem about AI"}
    ]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### Python - Conversation (Multi-turn)
```python
messages = []

# First turn
messages.append({
    "role": "user",
    "content": "What is machine learning?"
})

response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=messages
)

assistant_response = response.content[0].text
messages.append({
    "role": "assistant",
    "content": assistant_response
})

print(f"User: What is machine learning?")
print(f"Assistant: {assistant_response}\n")

# Second turn
messages.append({
    "role": "user",
    "content": "Can you explain deep learning?"
})

response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=messages
)

print(f"Assistant: {response.content[0].text}")
```

### TypeScript - Unary
```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const message = await client.messages.create({
  model: "claude-opus-4-8",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "Hello, Claude" }
  ]
});

console.log(message.content[0].type === "text" ? message.content[0].text : "");
```

### TypeScript - Streaming
```typescript
const stream = await client.messages.create({
  model: "claude-opus-4-8",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "Write a poem about AI" }
  ],
  stream: true
});

for await (const event of stream) {
  if (event.type === "content_block_delta" && event.delta.type === "text_delta") {
    process.stdout.write(event.delta.text);
  }
}
```

### TypeScript - Streaming with Helpers
```typescript
const stream = client.messages.stream({
  model: "claude-opus-4-8",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "Say hello!" }
  ]
});

// Use helpers
stream.on("text", (text) => {
  console.log(text);
});

const finalMessage = await stream.finalMessage();
console.log("Done!", finalMessage.usage);
```

### Common Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `model` | string | Model ID (see below) |
| `messages` | array | Message history with `role` and `content` |
| `max_tokens` | int | Maximum response length (required) |
| `system` | string | System prompt for model behavior |
| `temperature` | float | 0.0-1.0; higher = more creative (default: 1.0) |
| `top_p` | float | Nucleus sampling (0-1) |
| `top_k` | int | Top-K sampling |
| `timeout` | int | Request timeout in milliseconds |
| `stop_sequences` | list | Stop generation at these sequences |

---

## Available Models

| Model | Context | Cost | Best For |
|-------|---------|------|----------|
| `claude-opus-4-8` | 200K | High | Complex reasoning, analysis |
| `claude-sonnet-4-20250514` | 200K | Medium | Balanced speed/quality |
| `claude-haiku-4-8` | 200K | Low | Fast, lightweight tasks |
| `claude-3.5-sonnet` | 200K | Medium | Older version (not recommended) |

**Tip**: Use `claude-opus-4-8` for best performance, `claude-sonnet` for balance, `claude-haiku` for speed.

---

## Multimodal (Vision)

### Python - Image Input
```python
import anthropic
import base64

client = anthropic.Anthropic()

# From file
with open("image.jpg", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

message = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "image",
                    "source": {
                        "type": "base64",
                        "media_type": "image/jpeg",
                        "data": image_data,
                    },
                },
                {
                    "type": "text",
                    "text": "Describe this image"
                }
            ],
        }
    ],
)

print(message.content[0].text)
```

### Python - PDF Upload (Files API)
```python
import anthropic

client = anthropic.Anthropic()

# Upload PDF
with open("document.pdf", "rb") as f:
    response = client.beta.files.upload(file=f)
    file_id = response.id

# Use in message
message = client.beta.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "document",
                    "source": {
                        "type": "file",
                        "file_id": file_id
                    }
                },
                {
                    "type": "text",
                    "text": "Summarize this PDF"
                }
            ]
        }
    ],
    betas=["files-api-2025-04-14"]
)

# Clean up
client.beta.files.delete(file_id)
```

### TypeScript - Image Input
```typescript
import fs from "fs";

const imageBuffer = fs.readFileSync("image.jpg");
const base64Image = imageBuffer.toString("base64");

const message = await client.messages.create({
  model: "claude-opus-4-8",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: [
        {
          type: "image",
          source: {
            type: "base64",
            media_type: "image/jpeg",
            data: base64Image
          }
        },
        {
          type: "text",
          text: "Describe this image"
        }
      ]
    }
  ]
});

console.log(message.content[0].type === "text" ? message.content[0].text : "");
```

---

## Tool Use (Function Calling)

Enable Claude to call external functions.

### Python
```python
import anthropic
import json

client = anthropic.Anthropic()

tools = [
    {
        "name": "get_weather",
        "description": "Get the current weather in a location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "City and state/country"
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
]

messages = [
    {"role": "user", "content": "What's the weather in San Francisco?"}
]

response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    tools=tools,
    messages=messages
)

# Process tool calls
while response.stop_reason == "tool_use":
    tool_use_block = next(
        (block for block in response.content if block.type == "tool_use"),
        None
    )
    
    if tool_use_block:
        tool_name = tool_use_block.name
        tool_input = tool_use_block.input
        
        print(f"Tool: {tool_name}")
        print(f"Input: {tool_input}")
        
        # Execute tool
        if tool_name == "get_weather":
            tool_result = "Sunny, 72°F"
        else:
            tool_result = "Tool not found"
        
        # Continue conversation
        messages.append({"role": "assistant", "content": response.content})
        messages.append({
            "role": "user",
            "content": [
                {
                    "type": "tool_result",
                    "tool_use_id": tool_use_block.id,
                    "content": tool_result
                }
            ]
        })
        
        response = client.messages.create(
            model="claude-opus-4-8",
            max_tokens=1024,
            tools=tools,
            messages=messages
        )

# Get final response
final_text = next(
    (block.text for block in response.content if block.type == "text"),
    None
)
print(f"Final: {final_text}")
```

### TypeScript
```typescript
const tools: Anthropic.Tool[] = [
  {
    name: "get_weather",
    description: "Get the current weather in a location",
    input_schema: {
      type: "object",
      properties: {
        location: {
          type: "string",
          description: "City and state/country"
        },
        unit: {
          type: "string",
          enum: ["celsius", "fahrenheit"],
          description: "Temperature unit"
        }
      },
      required: ["location"]
    }
  }
];

const messages: Anthropic.MessageParam[] = [
  { role: "user", content: "What's the weather in San Francisco?" }
];

let response = await client.messages.create({
  model: "claude-opus-4-8",
  max_tokens: 1024,
  tools,
  messages
});

// Process tool calls
while (response.stop_reason === "tool_use") {
  const toolUseBlock = response.content.find(
    (block) => block.type === "tool_use"
  ) as Anthropic.ToolUseBlock | undefined;
  
  if (toolUseBlock) {
    console.log(`Tool: ${toolUseBlock.name}`);
    console.log(`Input: ${JSON.stringify(toolUseBlock.input)}`);
    
    let toolResult = "Sunny, 72°F";
    
    messages.push({ role: "assistant", content: response.content });
    messages.push({
      role: "user",
      content: [
        {
          type: "tool_result",
          tool_use_id: toolUseBlock.id,
          content: toolResult
        }
      ]
    });
    
    response = await client.messages.create({
      model: "claude-opus-4-8",
      max_tokens: 1024,
      tools,
      messages
    });
  }
}

const textBlock = response.content.find((block) => block.type === "text");
console.log(`Final: ${textBlock && textBlock.type === "text" ? textBlock.text : ""}`);
```

---

## Structured Outputs

Use schemas to ensure valid JSON responses.

### Python
```python
from typing import Optional
import anthropic
import json

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": "Extract name and age: John Smith, 28 years old"
        }
    ],
    extra_headers={
        "anthropic-beta": "structured-outputs-2025-04-14"
    }
)

# Parse response
response_text = message.content[0].text
data = json.loads(response_text)
print(f"Name: {data['name']}, Age: {data['age']}")
```

---

## Prompt Caching

Reduce costs by caching repeated content.

### Python
```python
import anthropic

client = anthropic.Anthropic()

# First request (establishes cache)
response1 = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=100,
    system=[
        {
            "type": "text",
            "text": "You are a helpful assistant."
        },
        {
            "type": "text",
            "text": "This is cached context that will be reused.",
            "cache_control": {"type": "ephemeral"}  # Cache this block
        }
    ],
    messages=[
        {"role": "user", "content": "Hello"}
    ]
)

print(f"Cache creation tokens: {response1.usage.cache_creation_input_tokens}")
print(f"Input tokens: {response1.usage.input_tokens}")

# Second request (uses cache)
response2 = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=100,
    system=[
        {
            "type": "text",
            "text": "You are a helpful assistant."
        },
        {
            "type": "text",
            "text": "This is cached context that will be reused.",
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[
        {"role": "user", "content": "How are you?"}
    ]
)

print(f"Cache read tokens: {response2.usage.cache_read_input_tokens}")
print(f"Input tokens: {response2.usage.input_tokens}")  # Much lower!
```

---

## Batch API

Process multiple messages efficiently at lower cost.

### Python
```python
import anthropic
import time

client = anthropic.Anthropic()

# Create batch requests
requests = [
    {
        "custom_id": "task-1",
        "params": {
            "model": "claude-opus-4-8",
            "max_tokens": 100,
            "messages": [
                {"role": "user", "content": "What is AI?"}
            ]
        }
    },
    {
        "custom_id": "task-2",
        "params": {
            "model": "claude-opus-4-8",
            "max_tokens": 100,
            "messages": [
                {"role": "user", "content": "What is ML?"}
            ]
        }
    }
]

# Submit batch
batch = client.beta.messages.batches.create(
    requests=requests
)

print(f"Batch ID: {batch.id}")
print(f"Status: {batch.processing_status}")

# Poll for completion
while batch.processing_status == "processing":
    time.sleep(5)
    batch = client.beta.messages.batches.retrieve(batch.id)
    print(f"Status: {batch.processing_status}")

# Get results
results = client.beta.messages.batches.results(batch.id)
for result in results:
    print(f"Task {result.custom_id}: {result.result.message.content[0].text}")
```

---

## Error Handling

### Python
```python
import anthropic
from anthropic import APIError, RateLimitError, APIConnectionError

try:
    message = client.messages.create(
        model="claude-opus-4-8",
        max_tokens=1024,
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
import Anthropic from "@anthropic-ai/sdk";

try {
  const message = await client.messages.create({
    model: "claude-opus-4-8",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello" }]
  });
} catch (error) {
  if (error instanceof Anthropic.RateLimitError) {
    console.error("Rate limited");
  } else if (error instanceof Anthropic.APIConnectionError) {
    console.error("Connection error");
  } else if (error instanceof Anthropic.APIError) {
    console.error(`API error: ${error.message}`);
  }
}
```

---

## Token Counting

Estimate costs before making requests.

### Python
```python
import anthropic

client = anthropic.Anthropic()

# Count tokens
count = client.messages.count_tokens(
    model="claude-opus-4-8",
    messages=[
        {"role": "user", "content": "Hello, Claude"}
    ]
)

print(f"Input tokens: {count.input_tokens}")

# With system prompt
count = client.messages.count_tokens(
    model="claude-opus-4-8",
    system="You are a helpful assistant.",
    messages=[
        {"role": "user", "content": "Explain machine learning"}
    ]
)

print(f"Total tokens: {count.input_tokens}")
```

---

## Usage & Token Management

### Python
```python
message = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}]
)

# Access token usage
print(f"Input tokens: {message.usage.input_tokens}")
print(f"Output tokens: {message.usage.output_tokens}")
print(f"Total: {message.usage.input_tokens + message.usage.output_tokens}")

# With caching
print(f"Cache creation: {message.usage.cache_creation_input_tokens}")
print(f"Cache read: {message.usage.cache_read_input_tokens}")
```

---

## Async Operations

### Python
```python
import asyncio
import anthropic

async def main():
    client = anthropic.AsyncAnthropic()
    
    message = await client.messages.create(
        model="claude-opus-4-8",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Hello"}]
    )
    
    print(message.content[0].text)

asyncio.run(main())
```

---

## Production Best Practices

1. **API Key Security**: Use environment variables; never hardcode keys
2. **Error Handling**: Catch rate limits and connection errors with retries
3. **Token Management**: Monitor token usage; plan for costs
4. **Streaming**: Use streaming for better UX with long responses
5. **Caching**: Enable prompt caching for repeated content to save costs
6. **Batching**: Use batch API for non-urgent, high-volume requests
7. **Model Selection**: Choose `opus` for quality, `sonnet` for balance, `haiku` for speed
8. **Timeout Handling**: Set appropriate timeouts for production
9. **Monitoring**: Log all API calls for debugging
10. **Rate Limiting**: Implement backoff strategies for 429 responses
11. **Tool Design**: Keep tool schemas simple and well-documented
12. **Max Tokens**: Always specify reasonable `max_tokens` limits

---

## Quick Reference

### Most Common Operations

```python
# Python: Basic message
import anthropic
client = anthropic.Anthropic()
message = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}]
)
print(message.content[0].text)

# Python: Streaming
with client.messages.stream(model="claude-opus-4-8", max_tokens=1024, 
                            messages=[{"role": "user", "content": "Write a poem"}]) as stream:
    for text in stream.text_stream:
        print(text, end="")

# Python: Tool use
response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "..."}]
)

# Python: Count tokens
count = client.messages.count_tokens(
    model="claude-opus-4-8",
    messages=[{"role": "user", "content": "..."}]
)
```

```typescript
// TypeScript: Basic message
import Anthropic from "@anthropic-ai/sdk";
const client = new Anthropic();
const message = await client.messages.create({
  model: "claude-opus-4-8",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello" }]
});

// TypeScript: Streaming
const stream = client.messages.stream({
  model: "claude-opus-4-8",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Write a poem" }]
});
stream.on("text", (text) => process.stdout.write(text));

// TypeScript: Tool use
const response = await client.messages.create({
  model: "claude-opus-4-8",
  max_tokens: 1024,
  tools,
  messages: [{ role: "user", content: "..." }]
});
```

### Response Structure
```json
{
  "id": "msg_123",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "response text"
    }
  ],
  "model": "claude-opus-4-8",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 10,
    "output_tokens": 20
  }
}
```

### Environment Setup
```bash
export ANTHROPIC_API_KEY="your-api-key"
```

---

## Links & Resources

- **Official Docs**: https://docs.anthropic.com
- **API Reference**: https://docs.anthropic.com/reference
- **Python SDK**: https://github.com/anthropics/anthropic-sdk-python
- **TypeScript SDK**: https://github.com/anthropics/anthropic-sdk-typescript
- **Console**: https://console.anthropic.com
- **Cookbook**: https://github.com/anthropics/anthropic-cookbook
- **Models**: https://docs.anthropic.com/en/docs/about/models
- **Discord**: https://discord.gg/anthropic

---

**Version**: SDK 0.36+ | **Updated**: June 2026