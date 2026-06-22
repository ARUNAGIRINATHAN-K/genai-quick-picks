# Groq SDK Reference

**Latest Update**: June 2026  
**Official Docs**: [console.groq.com/docs](https://console.groq.com/docs)  
**GitHub**: [groq/groq-python](https://github.com/groq/groq-python), [groq/groq-typescript](https://github.com/groq/groq-typescript)

## Installation

### Python
```bash
pip install groq
# Requires Python 3.7+
```

### TypeScript/JavaScript
```bash
npm install groq-sdk
# Requires TypeScript 4.9+ or Node.js 20+
```

---

## SDK Initialization

### Python
```python
import os
from groq import Groq

# Basic initialization
client = Groq(api_key=os.environ.get("GROQ_API_KEY"))

# API key can also be set via GROQ_API_KEY environment variable
# export GROQ_API_KEY="your-key"
client = Groq()

# With custom configuration
client = Groq(
    api_key=os.environ.get("GROQ_API_KEY"),
    timeout=30.0,  # Request timeout in seconds
    max_retries=2
)

# Async client
from groq import AsyncGroq
async_client = AsyncGroq(api_key=os.environ.get("GROQ_API_KEY"))
```

### TypeScript/JavaScript
```typescript
import Groq from "groq-sdk";

// Basic initialization
const groq = new Groq({
  apiKey: process.env.GROQ_API_KEY
});

// With custom configuration
const groq = new Groq({
  apiKey: process.env.GROQ_API_KEY,
  timeout: 30000,  // milliseconds
  maxRetries: 2
});

// Custom fetch client (for proxies, etc.)
const groq = new Groq({
  apiKey: process.env.GROQ_API_KEY,
  fetchOptions: {
    timeout: 30000
  }
});
```

---

## Authentication

Groq uses **API key authentication** via the `Authorization: Bearer` header.

```bash
# Set environment variable
export GROQ_API_KEY="your-api-key"
```

Get your API key from [Groq Console](https://console.groq.com/keys).

---

## Chat Completions (Core Generation)

Groq uses OpenAI-compatible API structure for familiarity.

### Python - Unary (Simple)
```python
from groq import Groq

client = Groq()

completion = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=[
        {
            "role": "system",
            "content": "You are a helpful assistant."
        },
        {
            "role": "user",
            "content": "Explain the importance of fast language models"
        }
    ],
    temperature=0.7,
    max_tokens=1024
)

print(completion.choices[0].message.content)
```

### Python - Streaming
```python
stream = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=[
        {"role": "user", "content": "Write a short poem about AI"}
    ],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### Python - Async
```python
import asyncio
from groq import AsyncGroq

async def main():
    client = AsyncGroq()
    
    completion = await client.chat.completions.create(
        model="llama-3.3-70b-versatile",
        messages=[
            {"role": "user", "content": "Hello!"}
        ]
    )
    
    print(completion.choices[0].message.content)

asyncio.run(main())
```

### TypeScript - Unary
```typescript
import Groq from "groq-sdk";

const groq = new Groq({
  apiKey: process.env.GROQ_API_KEY
});

async function main() {
  const completion = await groq.chat.completions.create({
    model: "llama-3.3-70b-versatile",
    messages: [
      {
        role: "system",
        content: "You are a helpful assistant."
      },
      {
        role: "user",
        content: "Explain the importance of fast language models"
      }
    ],
    temperature: 0.7,
    maxTokens: 1024
  });

  console.log(completion.choices[0].message.content);
}

main();
```

### TypeScript - Streaming
```typescript
const stream = await groq.chat.completions.create({
  model: "llama-3.3-70b-versatile",
  messages: [
    { role: "user", content: "Write a short poem about AI" }
  ],
  stream: true
});

for await (const chunk of stream) {
  if (chunk.choices[0].delta.content) {
    process.stdout.write(chunk.choices[0].delta.content);
  }
}
```

### Common Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `model` | string | Model identifier (see below) |
| `messages` | array | Message history with `role` and `content` |
| `temperature` | float | 0.0-2.0; higher = more creative |
| `max_tokens` | int | Maximum response length |
| `top_p` | float | Nucleus sampling (0-1) |
| `top_k` | int | Top-K sampling |
| `frequency_penalty` | float | -2.0 to 2.0 |
| `presence_penalty` | float | -2.0 to 2.0 |
| `repetition_penalty` | float | Repetition penalty |
| `seed` | int | Reproducibility |
| `stream` | bool | Enable streaming responses |

---

## Available Models

Groq specializes in **fast inference** on their LPU hardware.

| Model | Context | Tokens/Sec | Use Case |
|-------|---------|-----------|----------|
| `llama-3.3-70b-versatile` | 8K | ~300 | General-purpose, balanced |
| `llama-3.1-70b-versatile` | 128K | ~250 | Long context, reasoning |
| `llama-3.1-8b-instant` | 128K | ~400 | Fast, lightweight |
| `mixtral-8x7b-32768` | 32K | ~250 | Expert mixture, faster |
| `gemma-2-9b-it` | 8K | ~350 | Efficient instruction-tuned |

**Note**: Groq models are OpenAI-compatible; use standard OpenAI SDK if preferred.

---

## Multimodal (Vision)

Groq supports vision through OpenAI-compatible image inputs.

### Python
```python
import base64

with open("image.jpg", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

completion = client.chat.completions.create(
    model="llama-3.2-90b-vision-preview",  # Vision model
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Describe this image"},
                {
                    "type": "image_url",
                    "image_url": {"url": f"data:image/jpeg;base64,{image_data}"}
                }
            ]
        }
    ],
    max_tokens=1024
)

print(completion.choices[0].message.content)
```

### TypeScript
```typescript
import fs from "fs";

const imageBuffer = fs.readFileSync("image.jpg");
const base64Image = imageBuffer.toString("base64");

const completion = await groq.chat.completions.create({
  model: "llama-3.2-90b-vision-preview",
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Describe this image" },
        {
          type: "image_url",
          imageUrl: {
            url: `data:image/jpeg;base64,${base64Image}`
          }
        }
      ]
    }
  ],
  maxTokens: 1024
});

console.log(completion.choices[0].message.content);
```

---

## Tool/Function Calling

Enable models to call external functions.

### Python
```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_current_temperature",
            "description": "Gets the current temperature for a given location",
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

completion = client.chat.completions.create(
    model="llama-3.1-70b-versatile",
    messages=[
        {"role": "user", "content": "What's the weather in San Francisco?"}
    ],
    tools=tools,
    tool_choice="auto"
)

# Check for tool calls
if completion.choices[0].message.tool_calls:
    for tool_call in completion.choices[0].message.tool_calls:
        print(f"Function: {tool_call.function.name}")
        print(f"Arguments: {tool_call.function.arguments}")
        
        # Execute function
        if tool_call.function.name == "get_current_temperature":
            # Your implementation
            result = "72 degrees fahrenheit"
            
            # Send result back
            messages = [
                {"role": "user", "content": "What's the weather in San Francisco?"},
                completion.choices[0].message,
                {
                    "role": "tool",
                    "content": result,
                    "tool_call_id": tool_call.id
                }
            ]
            
            final = client.chat.completions.create(
                model="llama-3.1-70b-versatile",
                messages=messages,
                tools=tools
            )
            print(final.choices[0].message.content)
else:
    print("No tool calls requested")
```

### TypeScript
```typescript
const tools: Groq.Chat.CompletionTool[] = [
  {
    type: "function",
    function: {
      name: "get_current_temperature",
      description: "Gets the current temperature for a given location",
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

const completion = await groq.chat.completions.create({
  model: "llama-3.1-70b-versatile",
  messages: [
    { role: "user", content: "What's the weather in San Francisco?" }
  ],
  tools,
  toolChoice: "auto"
});

const toolCalls = completion.choices[0].message.toolCalls;
if (toolCalls) {
  for (const toolCall of toolCalls) {
    console.log(`Function: ${toolCall.function.name}`);
    console.log(`Arguments: ${toolCall.function.arguments}`);
    
    if (toolCall.function.name === "get_current_temperature") {
      const result = "72 degrees fahrenheit";
      
      const messages = [
        { role: "user" as const, content: "What's the weather in San Francisco?" },
        completion.choices[0].message,
        {
          role: "tool" as const,
          content: result,
          toolCallId: toolCall.id
        }
      ];
      
      const final = await groq.chat.completions.create({
        model: "llama-3.1-70b-versatile",
        messages,
        tools
      });
      console.log(final.choices[0].message.content);
    }
  }
}
```

---

## Structured Outputs (JSON Mode)

Force JSON responses without schema validation.

### Python
```python
completion = client.chat.completions.create(
    model="llama-3.1-70b-versatile",
    messages=[
        {
            "role": "user",
            "content": "Extract name and age: John Smith, 28 years old"
        }
    ],
    response_format={"type": "json_object"}
)

import json
result = json.loads(completion.choices[0].message.content)
print(f"Name: {result['name']}, Age: {result['age']}")
```

### TypeScript
```typescript
const completion = await groq.chat.completions.create({
  model: "llama-3.1-70b-versatile",
  messages: [
    {
      role: "user",
      content: "Extract name and age: John Smith, 28 years old"
    }
  ],
  responseFormat: { type: "json_object" }
});

const result = JSON.parse(completion.choices[0].message.content);
console.log(`Name: ${result.name}, Age: ${result.age}`);
```

---

## Speech-to-Text (Audio Transcription)

### Python
```python
import os

# Transcribe audio file
with open("sample_audio.m4a", "rb") as f:
    transcription = client.audio.transcriptions.create(
        file=(os.path.basename("sample_audio.m4a"), f),
        model="whisper-large-v3",
        language="en",  # Optional: specify language
        prompt="Specify context or technical terms",  # Optional
        response_format="json"  # Optional: "json", "text", "verbose_json"
    )

print(f"Transcription: {transcription.text}")
```

### TypeScript
```typescript
import fs from "fs";

const file = fs.createReadStream("sample_audio.m4a");
const transcription = await groq.audio.transcriptions.create({
  file: file,
  model: "whisper-large-v3",
  language: "en",
  responseFormat: "json"
});

console.log(`Transcription: ${transcription.text}`);
```

---

## Text-to-Speech

### Python
```python
# Generate speech from text
response = client.audio.speech.create(
    model="playai-tts",
    voice="Fritz-PlayAI",
    input="I love building and shipping new features!",
    response_format="wav"  # or "mp3"
)

# Save to file
response.write_to_file("output.wav")
```

### TypeScript
```typescript
import fs from "fs";

const response = await groq.audio.speech.create({
  model: "playai-tts",
  voice: "Fritz-PlayAI",
  input: "I love building and shipping new features!",
  responseFormat: "wav"
});

const buffer = Buffer.from(await response.arrayBuffer());
fs.writeFileSync("output.wav", buffer);
```

---

## Audio Translation

Translate audio to English.

### Python
```python
with open("sample_audio.m4a", "rb") as f:
    translation = client.audio.translations.create(
        file=(os.path.basename("sample_audio.m4a"), f),
        model="whisper-large-v3",
        response_format="json"
    )

print(f"Translation: {translation.text}")
```

### TypeScript
```typescript
const file = fs.createReadStream("sample_audio.m4a");
const translation = await groq.audio.translations.create({
  file: file,
  model: "whisper-large-v3",
  responseFormat: "json"
});

console.log(`Translation: ${translation.text}`);
```

---

## Error Handling

### Python
```python
from groq import Groq
from groq.exceptions import (
    APIConnectionError,
    RateLimitError,
    APIStatusError,
    APITimeoutError
)

try:
    completion = client.chat.completions.create(
        model="llama-3.3-70b-versatile",
        messages=[{"role": "user", "content": "Hello"}]
    )
except RateLimitError as e:
    print(f"Rate limited: {e}")
    # Implement exponential backoff
except APIConnectionError as e:
    print(f"Connection error: {e}")
except APITimeoutError as e:
    print(f"Timeout: {e}")
except APIStatusError as e:
    print(f"API error ({e.status_code}): {e}")
```

### TypeScript
```typescript
import Groq from "groq-sdk";

try {
  const completion = await groq.chat.completions.create({
    model: "llama-3.3-70b-versatile",
    messages: [{ role: "user", content: "Hello" }]
  });
} catch (error) {
  if (error instanceof Groq.RateLimitError) {
    console.error("Rate limited");
  } else if (error instanceof Groq.APIConnectionError) {
    console.error("Connection error");
  } else if (error instanceof Groq.APITimeoutError) {
    console.error("Timeout");
  } else if (error instanceof Groq.APIStatusError) {
    console.error(`API error: ${error.status}`);
  }
}
```

---

## Logging & Debugging

### Python
```python
import logging
from groq import Groq

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)

client = Groq()
completion = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=[{"role": "user", "content": "Hello"}]
)
```

### TypeScript
```typescript
const groq = new Groq({
  apiKey: process.env.GROQ_API_KEY,
  defaultHeaders: {
    "User-Agent": "my-app/1.0"
  }
});

// Log requests
groq.on("request", (request) => {
  console.log("Request:", request);
});

groq.on("response", (response) => {
  console.log("Response:", response);
});
```

---

## Advanced: Custom HTTP Client

### Python
```python
import httpx
from groq import Groq

# Create custom HTTP client
http_client = httpx.Client(
    timeout=30.0,
    limits=httpx.Limits(max_connections=100)
)

client = Groq(
    api_key="your-key",
    http_client=http_client
)
```

### TypeScript
```typescript
import Groq from "groq-sdk";

const groq = new Groq({
  apiKey: process.env.GROQ_API_KEY,
  fetch: async (url, init) => {
    // Custom fetch wrapper
    console.log(`Fetching ${url}`);
    return fetch(url, init);
  }
});
```

---

## Production Best Practices

1. **Rate Limiting**: Implement exponential backoff for 429 responses
2. **Timeouts**: Set reasonable timeouts (Groq is fast, but always prepare)
3. **Retries**: Auto-retry on connection errors but not on 4xx errors
4. **Error Categories**: Handle different error types distinctly
5. **Token Management**: Monitor token usage; Groq has rate limits per minute
6. **Model Selection**: Choose appropriate model for latency/quality trade-off
7. **Streaming**: Use streaming for better UX with real-time responses
8. **API Key Security**: Never expose keys in client-side code
9. **Logging**: Log request/response for debugging
10. **Health Checks**: Periodically test model availability
11. **Gradual Rollout**: Test with small traffic before full deployment
12. **Fallback Models**: Have backup models in case of unavailability

---

## Quick Reference

### Most Common Operations

```python
# Python: Basic chat
from groq import Groq
client = Groq()
completion = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=[{"role": "user", "content": "Hello"}]
)
print(completion.choices[0].message.content)

# Python: Streaming
stream = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=[...],
    stream=True
)
for chunk in stream:
    print(chunk.choices[0].delta.content or "", end="")

# Python: Async
import asyncio
from groq import AsyncGroq
async_client = AsyncGroq()
# Use await with same methods
```

```typescript
// TypeScript: Basic chat
import Groq from "groq-sdk";
const groq = new Groq();
const completion = await groq.chat.completions.create({
  model: "llama-3.3-70b-versatile",
  messages: [{ role: "user", content: "Hello" }]
});
console.log(completion.choices[0].message.content);

// TypeScript: Streaming
const stream = await groq.chat.completions.create({
  model: "llama-3.3-70b-versatile",
  messages: [...],
  stream: true
});
for await (const chunk of stream) {
  process.stdout.write(chunk.choices[0].delta.content || "");
}

// TypeScript: Audio
const transcription = await groq.audio.transcriptions.create({
  file: fs.createReadStream("audio.m4a"),
  model: "whisper-large-v3"
});
```

### Response Structure
```json
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion",
  "created": 1234567890,
  "model": "llama-3.3-70b-versatile",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "response text",
        "tool_calls": null
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
export GROQ_API_KEY="your-api-key"
```

---

## Links & Resources

- **Official Docs**: https://console.groq.com/docs
- **Quickstart**: https://console.groq.com/docs/quickstart
- **API Reference**: https://console.groq.com/docs/api-reference
- **Python SDK**: https://github.com/groq/groq-python
- **TypeScript SDK**: https://github.com/groq/groq-typescript
- **Console**: https://console.groq.com
- **Models**: https://console.groq.com/docs/models
- **Community**: https://community.groq.com

---

**Version**: SDK Latest | **Updated**: June 2026
