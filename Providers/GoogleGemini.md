# Google Gemini SDK Reference

**Latest Update**: June 2026  
**Official Docs**: [ai.google.dev](https://ai.google.dev/gemini-api/docs)  
**GitHub**: [googleapis/python-genai](https://github.com/googleapis/python-genai), [googleapis/js-genai](https://github.com/googleapis/js-genai)

## Installation

### Python
```bash
pip install google-genai
# Requires Python 3.9+
```

### TypeScript/JavaScript
```bash
npm install @google/genai
# Requires Node.js 18+
```

---

## SDK Initialization

### Python - Gemini API
```python
import os
from google import genai

# Basic initialization (Gemini Developer API)
client = genai.Client(api_key=os.environ["GOOGLE_API_KEY"])

# Alternative: set via environment
# export GOOGLE_API_KEY="your-key"
client = genai.Client()

# With custom configuration
client = genai.Client(
    api_key=os.environ["GOOGLE_API_KEY"],
    http_options={"timeout": 30}
)
```

### Python - Vertex AI (Google Cloud)
```python
import os
from google import genai

# Enterprise setup (requires Google Cloud Project)
client = genai.Client(
    vertexai=True,
    project="your-project-id",
    location="us-central1"
)

# Or via environment variables
# export GOOGLE_CLOUD_PROJECT=your-project
# export GOOGLE_CLOUD_LOCATION=us-central1
# export GOOGLE_GENAI_USE_ENTERPRISE=True
client = genai.Client(vertexai=True)
```

### TypeScript - Gemini API
```typescript
import { GoogleGenAI } from "@google/genai";

// Basic initialization
const ai = new GoogleGenAI({
  apiKey: process.env.GOOGLE_API_KEY
});

// With custom configuration
const ai = new GoogleGenAI({
  apiKey: process.env.GOOGLE_API_KEY,
  apiVersion: "v1beta"  // "v1" for stable API
});
```

### TypeScript - Vertex AI
```typescript
import { GoogleGenAI } from "@google/genai";

// Enterprise setup
const ai = new GoogleGenAI({
  vertexai: true,
  project: "your-project-id",
  location: "us-central1"
});

// With environment variables
// export GOOGLE_GENAI_USE_ENTERPRISE=true
// export GOOGLE_CLOUD_PROJECT=your-project
const ai = new GoogleGenAI({ vertexai: true });
```

---

## Authentication

Gemini supports multiple authentication methods:

### API Key (Development/Prototyping)
```bash
export GOOGLE_API_KEY="your-api-key"
# Get key from: https://ai.google.dev
```

### Service Account (Vertex AI/Production)
```bash
export GOOGLE_CLOUD_PROJECT="your-project"
export GOOGLE_CLOUD_LOCATION="us-central1"
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account.json"
```

---

## Chat Completions (Core Generation)

### Python - Simple Generation
```python
from google import genai

client = genai.Client(api_key="your-key")

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What is machine learning?"
)

print(response.text)
```

### Python - Streaming
```python
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Explain quantum computing",
    stream=True
)

for chunk in response:
    if chunk.text:
        print(chunk.text, end="")
```

### Python - Conversation (Multi-turn)
```python
client = genai.Client()

# Create a chat session
chat = client.chats.create(
    model="gemini-2.5-flash",
    config=genai.types.GenerateContentConfig(
        temperature=0.7,
        max_output_tokens=1024
    )
)

# First turn
response1 = chat.send_message("What is AI?")
print(response1.text)

# Second turn (context maintained)
response2 = chat.send_message("Can you explain more?")
print(response2.text)

# View full history
print(chat.history)
```

### TypeScript - Simple Generation
```typescript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({
  apiKey: process.env.GOOGLE_API_KEY
});

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "What is machine learning?"
});

console.log(response.text);
```

### TypeScript - Streaming
```typescript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Explain quantum computing",
  stream: true
});

for await (const chunk of response) {
  if (chunk.text) {
    process.stdout.write(chunk.text);
  }
}
```

### TypeScript - Conversation
```typescript
const chat = await ai.chats.create({
  model: "gemini-2.5-flash",
  config: {
    temperature: 0.7,
    maxOutputTokens: 1024
  }
});

// First turn
const response1 = await chat.sendMessage("What is AI?");
console.log(response1.text);

// Second turn
const response2 = await chat.sendMessage("Can you explain more?");
console.log(response2.text);

// View history
console.log(chat.history);
```

### Common Generation Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `temperature` | float | 0.0-2.0; higher = more random (default: 1.0) |
| `max_output_tokens` | int | Max response length |
| `top_p` | float | Nucleus sampling (0-1) |
| `top_k` | int | Top-K sampling |
| `frequency_penalty` | float | -2.0 to 2.0; penalizes repetition |
| `presence_penalty` | float | -2.0 to 2.0; encourages new topics |
| `seed` | int | For reproducibility |
| `response_mime_type` | string | `text/plain` or `application/json` |

---

## Available Models

| Model | Context | Capabilities | Best For |
|-------|---------|--------------|----------|
| `gemini-2.5-flash` | 1M | Fast, multimodal | General-purpose, real-time |
| `gemini-2.5-pro` | 1M | Advanced reasoning | Complex tasks, analysis |
| `gemini-3.5-flash-lite-preview` | Varies | Ultra-fast | High-volume, low-latency |
| `gemini-3.1-pro-preview` | 1M | Expert-level reasoning | Research, coding |

---

## Multimodal Input (Vision, Audio, Video, Files)

### Python - Image Input
```python
from pathlib import Path
from google import genai

client = genai.Client()

# From file
image_part = genai.upload_file(path="path/to/image.jpg")

# From URL
image_url = {
    "inline_data": {
        "mime_type": "image/jpeg",
        "data": base64_image_data  # base64 encoded
    }
}

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        "Analyze this image:",
        image_part
    ]
)

print(response.text)

# Clean up
import os
os.remove(image_part.name)
```

### Python - PDF/Document Upload
```python
# Upload document (up to 1000 pages)
pdf_file = genai.upload_file(
    path="large_document.pdf",
    mime_type="application/pdf"
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        "Summarize this document in 3 bullet points",
        pdf_file
    ]
)

print(response.text)
```

### Python - Video Upload
```python
# Upload video for analysis
video_file = genai.upload_file(
    path="sample.mp4",
    mime_type="video/mp4"
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        "Describe what happens in this video",
        video_file
    ]
)

print(response.text)
```

### TypeScript - Image Input
```typescript
import fs from "fs";

const ai = new GoogleGenAI();

// From file with base64
const imageData = fs.readFileSync("image.jpg");
const base64Image = imageData.toString("base64");

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    "Analyze this image:",
    {
      inlineData: {
        mimeType: "image/jpeg",
        data: base64Image
      }
    }
  ]
});

console.log(response.text);
```

---

## Tool/Function Calling

Enable models to call external functions and APIs.

### Python
```python
from google import genai
from google.genai import types

client = genai.Client()

# Define tools
tools = [
    types.Tool(
        function_declarations=[
            types.FunctionDeclaration(
                name="get_weather",
                description="Get weather for a location",
                parameters=types.Schema(
                    type="OBJECT",
                    properties={
                        "location": types.Schema(type="STRING"),
                        "unit": types.Schema(
                            type="STRING",
                            enum=["celsius", "fahrenheit"]
                        )
                    },
                    required=["location"]
                )
            )
        ]
    )
]

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What's the weather in Paris?",
    tools=tools
)

# Process function calls
if response.tool_calls:
    for tool_call in response.tool_calls[0].function_calls:
        print(f"Called: {tool_call.name}")
        print(f"Arguments: {tool_call.args}")
        
        # Execute function
        if tool_call.name == "get_weather":
            result = "Sunny, 20°C"
            
            # Send result back
            response = client.models.generate_content(
                model="gemini-2.5-flash",
                contents=[
                    "What's the weather in Paris?",
                    response,
                    types.Content(
                        role="user",
                        parts=[
                            types.Part.from_function_response(
                                name="get_weather",
                                response={"result": result}
                            )
                        ]
                    )
                ],
                tools=tools
            )
            print(response.text)
```

### TypeScript
```typescript
const tools = [
  {
    functionDeclarations: [
      {
        name: "get_weather",
        description: "Get weather for a location",
        parameters: {
          type: "OBJECT",
          properties: {
            location: { type: "STRING" },
            unit: {
              type: "STRING",
              enum: ["celsius", "fahrenheit"]
            }
          },
          required: ["location"]
        }
      }
    ]
  }
];

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "What's the weather in Paris?",
  tools
});

// Process tool calls
if (response.toolCalls) {
  for (const toolCall of response.toolCalls) {
    console.log(`Called: ${toolCall.name}`);
    console.log(`Arguments: ${JSON.stringify(toolCall.args)}`);
    
    if (toolCall.name === "get_weather") {
      const result = "Sunny, 20°C";
      
      // Send result back
      const finalResponse = await ai.models.generateContent({
        model: "gemini-2.5-flash",
        contents: [
          "What's the weather in Paris?",
          response,
          {
            role: "user",
            parts: [
              {
                functionResponse: {
                  name: "get_weather",
                  response: { result }
                }
              }
            ]
          }
        ],
        tools
      });
      console.log(finalResponse.text);
    }
  }
}
```

---

## Structured Outputs (JSON Mode)

Force responses to conform to a JSON schema.

### Python
```python
from google import genai
from google.genai import types

client = genai.Client()

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Extract person details from: John Smith, 28 years old",
    config=genai.types.GenerateContentConfig(
        response_mime_type="application/json",
        response_schema=types.Schema(
            type="OBJECT",
            properties={
                "name": types.Schema(type="STRING"),
                "age": types.Schema(type="INTEGER")
            },
            required=["name", "age"]
        )
    )
)

import json
data = json.loads(response.text)
print(f"Name: {data['name']}, Age: {data['age']}")
```

### TypeScript
```typescript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Extract person details from: John Smith, 28 years old",
  config: {
    responseMimeType: "application/json",
    responseSchema: {
      type: "OBJECT",
      properties: {
        name: { type: "STRING" },
        age: { type: "INTEGER" }
      },
      required: ["name", "age"]
    }
  }
});

const data = JSON.parse(response.text);
console.log(`Name: ${data.name}, Age: ${data.age}`);
```

---

## Embeddings

Generate vector embeddings for semantic search and RAG.

### Python
```python
from google import genai

client = genai.Client()

response = client.models.embed_content(
    model="text-embedding-004",
    content="This is a sample sentence."
)

embedding = response.embedding
print(f"Embedding dimension: {len(embedding)}")
print(f"First 5 values: {embedding[:5]}")

# Batch embeddings
response = client.models.embed_content(
    model="text-embedding-004",
    content=[
        "First text",
        "Second text",
        "Third text"
    ]
)

for i, embedding in enumerate(response.embeddings):
    print(f"Text {i}: {len(embedding)} dimensions")
```

### TypeScript
```typescript
const response = await ai.models.embedContent({
  model: "text-embedding-004",
  content: "This is a sample sentence."
});

const embedding = response.embedding;
console.log(`Embedding dimension: ${embedding.length}`);
console.log(`First 5 values: ${embedding.slice(0, 5)}`);

// Batch embeddings
const batchResponse = await ai.models.embedContent({
  model: "text-embedding-004",
  content: [
    "First text",
    "Second text",
    "Third text"
  ]
});

batchResponse.embeddings.forEach((emb, i) => {
  console.log(`Text ${i}: ${emb.length} dimensions`);
});
```

### Embedding Models

| Model | Dimension | Use Case |
|-------|-----------|----------|
| `text-embedding-004` | 768 | General-purpose semantic search |

---

## Live API (Real-time Audio/Video)

Two-way streaming for real-time interaction.

### Python
```python
from google import genai
import time

client = genai.Client()

# Create session
config = genai.types.LiveConnectConfig(
    model="gemini-2.5-live"
)

session = client.agentic.sessions.create(config=config)

# Send audio frame
session.send(
    genai.types.RealtimeInput(
        media_stream=genai.types.MediaStreamInput(
            audio_chunks=[audio_data]  # bytes
        )
    )
)

# Receive response
for response in session.receive():
    if response.server_content.model_turn:
        for part in response.server_content.model_turn.parts:
            if part.text:
                print(part.text)
            elif part.audio_bytes:
                # Handle audio response
                pass

session.close()
```

---

## Thinking/Extended Thinking

Let models work through complex problems with extended reasoning.

### Python
```python
from google import genai

client = genai.Client()

response = client.models.generate_content(
    model="gemini-2.5-pro",  # Pro model required
    contents="Solve: 2x^2 - 4x + 2 = 0",
    config=genai.types.GenerateContentConfig(
        thinking=genai.types.ThinkingConfig(
            budget_tokens=10000  # Allow deep thinking
        )
    )
)

# Response includes thinking process
for part in response.parts:
    if part.thinking:
        print(f"Thinking: {part.thinking}")
    elif part.text:
        print(f"Answer: {part.text}")
```

---

## Batch Processing (Large-scale)

### Python
```python
from google import genai
import time

client = genai.Client()

# Create batch request
requests = [
    genai.types.Content(parts=[
        {"text": "What is quantum computing?"}
    ]),
    genai.types.Content(parts=[
        {"text": "Explain machine learning"}
    ]),
    genai.types.Content(parts=[
        {"text": "What is AI?"}
    ])
]

batch = client.batches.create(
    requests=[
        {"contents": [req]} for req in requests
    ]
)

# Poll for completion
while batch.state == "PROCESSING":
    print("Processing...")
    time.sleep(5)
    batch = client.batches.get(batch.name)

# Get results
if batch.state == "COMPLETED":
    for result in batch.results:
        print(result)
```

---

## Error Handling

### Python
```python
from google import genai
from google.genai.exceptions import (
    ClientError,
    APIConnectionError,
    InvalidArgumentError
)

try:
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents="Hello"
    )
except InvalidArgumentError as e:
    print(f"Invalid argument: {e}")
except APIConnectionError as e:
    print(f"Connection error: {e}")
except ClientError as e:
    print(f"Client error: {e}")
```

### TypeScript
```typescript
import { GoogleGenAIError } from "@google/genai";

try {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: "Hello"
  });
} catch (e) {
  if (e instanceof GoogleGenAIError) {
    console.error(`API Error: ${e.message}`);
    console.error(`Code: ${e.code}`);
  } else {
    console.error(`Unknown error: ${e}`);
  }
}
```

---

## File Management

### Python
```python
from google import genai
import time

client = genai.Client()

# Upload file
file = genai.upload_file(path="document.pdf")
print(f"File uploaded: {file.name}")
print(f"MIME type: {file.mime_type}")

# List files
for f in client.files.list():
    print(f.name)

# Get file info
file_info = client.files.get(file.name)
print(f"Size: {file_info.size_bytes} bytes")
print(f"Created: {file_info.create_time}")

# Delete file
client.files.delete(file.name)
print("File deleted")

# Wait for processing (files need to be ready before use)
max_retries = 10
for _ in range(max_retries):
    if file.state == "ACTIVE":
        break
    time.sleep(5)
    file = client.files.get(file.name)
```

---

## Production Best Practices

1. **API Key Security**: Never embed keys in client-side code; use server proxies
2. **Rate Limiting**: Implement backoff for rate limit errors (429)
3. **Timeout Handling**: Set appropriate timeouts for long-running operations
4. **Error Recovery**: Implement retry logic with exponential backoff
5. **Token Management**: Monitor input/output tokens; be aware of context window limits
6. **Model Selection**: Choose appropriate models for your use case (Flash for speed, Pro for reasoning)
7. **Batch Processing**: Use batches API for large-scale, non-time-sensitive requests
8. **File Cleanup**: Delete uploaded files when no longer needed
9. **Caching**: Cache embeddings and responses when appropriate
10. **Monitoring**: Log all API calls for debugging and audit purposes
11. **Vertex AI for Production**: Use Vertex AI instead of Developer API for production workloads
12. **Data Privacy**: Keep sensitive data off-device when possible

---

## Quick Reference

### Most Common Operations

```python
# Python: Basic generation
from google import genai
client = genai.Client(api_key="key")
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Hello!"
)
print(response.text)

# Python: Streaming
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Tell a story",
    stream=True
)
for chunk in response:
    print(chunk.text, end="")

# Python: Embeddings
embeddings = client.models.embed_content(
    model="text-embedding-004",
    content=["text1", "text2"]
)
```

```typescript
// TypeScript: Basic generation
import { GoogleGenAI } from "@google/genai";
const ai = new GoogleGenAI({ apiKey: "key" });
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Hello!"
});
console.log(response.text);

// TypeScript: Streaming
const stream = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Tell a story",
  stream: true
});
for await (const chunk of stream) {
  process.stdout.write(chunk.text || "");
}

// TypeScript: Embeddings
const emb = await ai.models.embedContent({
  model: "text-embedding-004",
  content: ["text1", "text2"]
});
```

### Response Structure
```python
{
    "text": "Response content",
    "parts": [
        {"text": "..."},
        {"inlineData": {...}},
        {"functionCall": {...}}
    ],
    "toolCalls": [...],
    "usageMetadata": {
        "promptTokenCount": 10,
        "candidatesTokenCount": 20,
        "totalTokenCount": 30
    }
}
```

### Environment Variables
```bash
# Gemini API
export GOOGLE_API_KEY="your-api-key"

# Vertex AI
export GOOGLE_CLOUD_PROJECT="your-project"
export GOOGLE_CLOUD_LOCATION="us-central1"
export GOOGLE_GENAI_USE_ENTERPRISE=true
```

---

## Links & Resources

- **Official Docs**: https://ai.google.dev/gemini-api/docs
- **Python SDK Repo**: https://github.com/googleapis/python-genai
- **TypeScript SDK Repo**: https://github.com/googleapis/js-genai
- **Google AI Studio**: https://ai.google.dev
- **Vertex AI**: https://cloud.google.com/vertex-ai
- **Model Cards**: https://ai.google.dev/models
- **Cookbook**: https://github.com/google-gemini/cookbook
- **Discord Community**: https://discord.gg/google-ai-dev

---

**Version**: SDK v0.2+ (GenAI) | **Updated**: June 2026
