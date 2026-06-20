# 🐍 OpenAI Python SDK

> `pip install openai` · Python 3.9+

---

## Client Initialization

```python
from openai import OpenAI
client = OpenAI()
# Reads OPENAI_API_KEY from env by default

from openai import OpenAI
client = OpenAI(api_key="sk-...")
# Explicit API key

from openai import AsyncOpenAI
client = AsyncOpenAI()
# Async client — use with await

from openai import AzureOpenAI
client = AzureOpenAI(
    azure_endpoint="https://YOUR_RESOURCE.openai.azure.com/",
    api_version="2025-01-01",
    api_key="..."
)
# Azure-hosted models

client = OpenAI(timeout=20.0, max_retries=3)
# Custom timeout and retry settings

client2 = client.with_options(timeout=5.0, max_retries=0)
# Per-call override without modifying the original client
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `OpenAI` | `client = OpenAI()` | Sync client — reads `OPENAI_API_KEY` from environment automatically |
| `OpenAI` | `client = OpenAI(api_key="sk-...")` | Sync client with an explicit API key passed inline |
| `AsyncOpenAI` | `client = AsyncOpenAI()` | Async client — every method must be called with `await` |
| `AzureOpenAI` | `client = AzureOpenAI(azure_endpoint=..., api_version=...)` | Client for models hosted on Azure OpenAI Service |
| `OpenAI` | `client = OpenAI(timeout=20.0, max_retries=3)` | Set default timeout (seconds) and retry count for all requests |
| `client.with_options` | `client2 = client.with_options(timeout=5.0, max_retries=0)` | Return a modified copy of the client without altering the original |

---

## Chat Completions

```python
completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is Python?"}
    ]
)
print(completion.choices[0].message.content)
# Basic chat completion

completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    temperature=0.7,
    max_tokens=512,
    top_p=1,
    frequency_penalty=0,
    presence_penalty=0
)
# With generation parameters

completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    n=3
)
# Return multiple response choices

completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    logprobs=True,
    top_logprobs=5
)
# Return log probabilities for tokens

completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    stop=["END", "\n\n"]
)
# Stop generation at specific sequences
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.chat.completions.create` | `completion = client.chat.completions.create(model="gpt-4o", messages=[...])` | Core call — send a messages array and receive a completion response |
| `temperature` | `client.chat.completions.create(..., temperature=0.7)` | Control randomness — lower is focused, higher is creative (range 0–2) |
| `max_tokens` | `client.chat.completions.create(..., max_tokens=512)` | Hard cap on the number of tokens the model can generate in a response |
| `top_p` | `client.chat.completions.create(..., top_p=0.9)` | Nucleus sampling threshold — alternative to `temperature` for diversity |
| `frequency_penalty` | `client.chat.completions.create(..., frequency_penalty=0.5)` | Reduce repetition of tokens that have already appeared (range 0–2) |
| `presence_penalty` | `client.chat.completions.create(..., presence_penalty=0.5)` | Encourage the model to introduce new topics rather than repeat them (range 0–2) |
| `n` | `client.chat.completions.create(..., n=3)` | Return `n` independent completion choices in a single response |
| `logprobs` | `client.chat.completions.create(..., logprobs=True, top_logprobs=5)` | Include log probabilities for the top-N tokens at each output position |
| `stop` | `client.chat.completions.create(..., stop=["END", "\n\n"])` | Stop generation as soon as any of the specified sequences are produced |
| `completion.choices[0].message.content` | `text = completion.choices[0].message.content` | Extract the text of the first generated message from the response |

---

## Structured Outputs

```python
from pydantic import BaseModel

class Reply(BaseModel):
    answer: str
    confidence: float

result = client.chat.completions.parse(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What is 2+2?"}],
    response_format=Reply
)
print(result.choices[0].message.parsed.answer)
# Parse response directly into a Pydantic model

completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    response_format={"type": "json_object"}
)
# Force the model to output valid JSON
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.chat.completions.parse` | `result = client.chat.completions.parse(model=..., messages=..., response_format=Reply)` | Generate a response and auto-parse it into a typed Pydantic model |
| `result.choices[0].message.parsed` | `obj = result.choices[0].message.parsed` | Access the parsed Pydantic object directly from the response |
| `response_format` | `client.chat.completions.create(..., response_format={"type": "json_object"})` | Guarantee the model returns a valid JSON object — parse it yourself |
| `response_format` | `client.chat.completions.create(..., response_format={"type": "json_schema", "json_schema": {...}})` | Constrain output to a specific JSON schema definition |

---

## Responses API

```python
response = client.responses.create(
    model="gpt-4.1",
    input="Explain recursion."
)
print(response.output_text)
# Simple text response

response = client.responses.create(
    model="gpt-4.1",
    input=[{"role": "user", "content": "Summarize this."}],
    instructions="You are a concise summarizer."
)
# With system instructions and message array

response = client.responses.retrieve("resp_abc123")
# Fetch a previously stored response

client.responses.delete("resp_abc123")
# Delete a stored response

page = client.responses.list()
for r in page:
    print(r.id)
# List all responses
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.responses.create` | `response = client.responses.create(model="gpt-4.1", input="Hello")` | Primary API — generate a response from a string or message array |
| `client.responses.create` | `response = client.responses.create(..., instructions="You are a summarizer.")` | Set a system-level instruction without adding a system message manually |
| `response.output_text` | `text = response.output_text` | Shortcut to extract plain text without traversing the output blocks array |
| `client.responses.retrieve` | `response = client.responses.retrieve("resp_abc123")` | Fetch a previously created and stored response by its ID |
| `client.responses.delete` | `client.responses.delete("resp_abc123")` | Permanently delete a stored response |
| `client.responses.list` | `page = client.responses.list()` | List all responses for your organization, paginated |
| `client.responses.input_items.list` | `items = client.responses.input_items.list("resp_abc123")` | List the exact input items submitted with a specific response |

---

## Streaming

```python
with client.chat.completions.stream(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Tell me a story."}]
) as stream:
    for chunk in stream:
        print(chunk.choices[0].delta.content or "", end="")
# Stream chat completions chunk by chunk

with client.chat.completions.stream(...) as stream:
    final = stream.get_final_completion()
# Get the fully assembled completion after streaming

with client.responses.stream(
    model="gpt-4.1",
    input="Write a poem."
) as stream:
    for event in stream:
        print(event)
# Stream a Responses API call

stream = client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    stream=True
)
for chunk in stream:
    print(chunk.choices[0].delta.content or "", end="")
# Raw stream using create() directly
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.chat.completions.stream` | `with client.chat.completions.stream(...) as stream:` | Managed streaming context — yields typed events as tokens arrive |
| `client.responses.stream` | `with client.responses.stream(...) as stream:` | Managed streaming for the Responses API — yields server-sent events |
| `client.chat.completions.create` | `stream = client.chat.completions.create(..., stream=True)` | Raw streaming mode — yields delta chunks without a context manager |
| `chunk.choices[0].delta.content` | `text = chunk.choices[0].delta.content or ""` | Extract the incremental text token from each raw streaming chunk |
| `stream.get_final_completion` | `final = stream.get_final_completion()` | Assemble and return the full completion object after the stream ends |
| `stream.get_final_message` | `msg = stream.get_final_message()` | Return the complete assembled message after a chat stream ends |

---

## Async Usage

```python
import asyncio
from openai import AsyncOpenAI

client = AsyncOpenAI()

async def main():
    completion = await client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": "Hello!"}]
    )
    print(completion.choices[0].message.content)

asyncio.run(main())
# Async chat completion

async def stream_async():
    async with client.chat.completions.stream(
        model="gpt-4o",
        messages=[{"role": "user", "content": "Count to 5."}]
    ) as stream:
        async for chunk in stream:
            print(chunk.choices[0].delta.content or "", end="")
# Async streaming
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `AsyncOpenAI` | `client = AsyncOpenAI()` | Drop-in async version of the sync client — same methods, all awaitable |
| `client.chat.completions.create` | `completion = await client.chat.completions.create(model=..., messages=...)` | Async chat completion — non-blocking, must run inside an event loop |
| `client.chat.completions.stream` | `async with client.chat.completions.stream(...) as stream:` | Async streaming context manager for chat completions |
| `async for chunk in stream` | `async for chunk in stream: print(chunk.choices[0].delta.content)` | Iterate over streaming chunks asynchronously |
| `asyncio.run` | `asyncio.run(main())` | Entry point to run a top-level async function from synchronous code |

---

## Tool / Function Calling

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the weather for a city.",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string"}
                },
                "required": ["city"]
            }
        }
    }
]

completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What's the weather in Tokyo?"}],
    tools=tools,
    tool_choice="auto"
)
# Let the model decide whether to call a tool

completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    tools=tools,
    tool_choice={"type": "function", "function": {"name": "get_weather"}}
)
# Force a specific tool call

tool_call = completion.choices[0].message.tool_calls[0]
print(tool_call.function.name)
print(tool_call.function.arguments)
# Extract tool call name and arguments from response

messages.append(completion.choices[0].message)
messages.append({
    "role": "tool",
    "tool_call_id": tool_call.id,
    "content": '{"temp": "15C", "condition": "Cloudy"}'
})
# Send tool result back to the model
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `tools` | `client.chat.completions.create(..., tools=[{"type": "function", "function": {...}}])` | Define one or more callable functions the model can invoke |
| `tool_choice` | `client.chat.completions.create(..., tool_choice="auto")` | Let the model decide whether and which tool to call |
| `tool_choice` | `client.chat.completions.create(..., tool_choice="none")` | Disable tool calling entirely for this request |
| `tool_choice` | `client.chat.completions.create(..., tool_choice={"type": "function", "function": {"name": "fn"}})` | Force the model to call a specific named function |
| `completion.choices[0].message.tool_calls` | `calls = completion.choices[0].message.tool_calls` | List of tool calls the model wants to make — may be multiple |
| `tool_call.function.name` | `name = tool_call.function.name` | Name of the function the model chose to invoke |
| `tool_call.function.arguments` | `args = tool_call.function.arguments` | JSON string of arguments the model passed to the function |
| `role: "tool"` | `messages.append({"role": "tool", "tool_call_id": ..., "content": ...})` | Return a tool result back into the conversation to continue generation |

---

## Vision (Image Input)

```python
completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What is in this image?"},
                {"type": "image_url", "image_url": {"url": "https://example.com/photo.jpg"}}
            ]
        }
    ]
)
# Send a public image URL

import base64

with open("image.png", "rb") as f:
    b64 = base64.b64encode(f.read()).decode()

completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Describe this image."},
                {"type": "image_url", "image_url": {"url": f"data:image/png;base64,{b64}"}}
            ]
        }
    ]
)
# Send a local image as base64
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `image_url` content block | `{"type": "image_url", "image_url": {"url": "https://..."}}` | Pass a publicly accessible image URL directly in the message content |
| `image_url` (base64) | `{"type": "image_url", "image_url": {"url": f"data:image/png;base64,{b64}"}}` | Pass a local image encoded as a base64 data URI |
| `text` content block | `{"type": "text", "text": "Describe this."}` | Combine a text prompt with one or more image blocks in the same message |
| `image_url.detail` | `{"image_url": {"url": "...", "detail": "high"}}` | Set to `"low"`, `"high"`, or `"auto"` to control resolution and token cost |

---

## Embeddings

```python
response = client.embeddings.create(
    input="The quick brown fox",
    model="text-embedding-3-small"
)
vector = response.data[0].embedding
# Single string embedding

response = client.embeddings.create(
    input=["sentence one", "sentence two", "sentence three"],
    model="text-embedding-3-small"
)
vectors = [d.embedding for d in response.data]
# Batch embedding for multiple strings

response = client.embeddings.create(
    input="Hello",
    model="text-embedding-3-large",
    dimensions=512
)
# Reduce output dimensions (for storage efficiency)
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.embeddings.create` | `response = client.embeddings.create(input="Hello", model="text-embedding-3-small")` | Generate a vector embedding for a single string input |
| `client.embeddings.create` | `response = client.embeddings.create(input=["a", "b", "c"], model=...)` | Batch input — returns one embedding vector per string in a single call |
| `dimensions` | `client.embeddings.create(..., dimensions=512)` | Truncate the output vector to a smaller size to save storage and compute |
| `response.data[0].embedding` | `vector = response.data[0].embedding` | Access the raw float vector from the first embedding in the response |
| `response.usage.total_tokens` | `tokens = response.usage.total_tokens` | Check how many tokens were consumed by the embedding request |

---

## Images (DALL-E)

```python
image = client.images.generate(
    prompt="A red fox sitting on a snowy mountain",
    model="dall-e-3",
    n=1,
    size="1024x1024"
)
print(image.data[0].url)
# Generate an image from a prompt

image = client.images.generate(
    prompt="A futuristic city",
    model="dall-e-3",
    response_format="b64_json"
)
# Return image as base64 instead of URL

image = client.images.edit(
    image=open("original.png", "rb"),
    mask=open("mask.png", "rb"),
    prompt="Add a sunset in the background",
    n=1,
    size="1024x1024"
)
# Edit an image with a mask (transparent areas get filled)

image = client.images.create_variation(
    image=open("original.png", "rb"),
    n=2,
    size="1024x1024"
)
# Generate variations of an existing image
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.images.generate` | `image = client.images.generate(prompt="A fox", model="dall-e-3", size="1024x1024")` | Generate one or more images from a text prompt using DALL-E |
| `response_format` | `client.images.generate(..., response_format="url")` | Return a temporary hosted URL (default) — expires after 60 minutes |
| `response_format` | `client.images.generate(..., response_format="b64_json")` | Return the image as a base64-encoded string for immediate local use |
| `quality` | `client.images.generate(..., quality="hd")` | Render at higher detail — DALL-E 3 only, increases cost |
| `style` | `client.images.generate(..., style="vivid")` | Set rendering style — DALL-E 3 only: `"vivid"` or `"natural"` |
| `client.images.edit` | `image = client.images.edit(image=..., mask=..., prompt="Add sunset")` | Modify an image by filling in transparent mask areas guided by a prompt |
| `client.images.create_variation` | `image = client.images.create_variation(image=open("img.png","rb"), n=2)` | Generate `n` variations of an existing image without a prompt |
| `image.data[0].url` | `url = image.data[0].url` | URL of the first generated image in the response |

---

## Audio

```python
response = client.audio.speech.create(
    model="tts-1",
    voice="alloy",
    input="Hello, this is a text to speech test."
)
response.stream_to_file("output.mp3")
# Text to speech — save to file

with client.audio.speech.with_streaming_response.create(
    model="tts-1",
    voice="nova",
    input="Streaming audio output."
) as response:
    response.stream_to_file("output.mp3")
# Streaming TTS to avoid buffering large audio

with open("audio.mp3", "rb") as f:
    transcript = client.audio.transcriptions.create(
        model="whisper-1",
        file=f
    )
print(transcript.text)
# Transcribe audio to text (Whisper)

with open("audio.mp3", "rb") as f:
    transcript = client.audio.transcriptions.create(
        model="whisper-1",
        file=f,
        response_format="verbose_json",
        timestamp_granularities=["word"]
    )
# Transcription with word-level timestamps

with open("audio.mp3", "rb") as f:
    translation = client.audio.translations.create(
        model="whisper-1",
        file=f
    )
print(translation.text)
# Translate foreign audio to English
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.audio.speech.create` | `response = client.audio.speech.create(model="tts-1", voice="alloy", input="Hello")` | Convert text to spoken audio and return streamable binary audio |
| `voice` | `client.audio.speech.create(..., voice="nova")` | Choose a TTS voice — options: `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer` |
| `response.stream_to_file` | `response.stream_to_file("output.mp3")` | Write the returned audio directly to a local file |
| `client.audio.speech.with_streaming_response.create` | `with client.audio.speech.with_streaming_response.create(...) as r:` | Stream audio bytes as they arrive — avoids loading large files into memory |
| `client.audio.transcriptions.create` | `transcript = client.audio.transcriptions.create(file=f, model="whisper-1")` | Transcribe an audio file to text using the Whisper model |
| `timestamp_granularities` | `client.audio.transcriptions.create(..., timestamp_granularities=["word"])` | Include per-word start/end timestamps in the transcription output |
| `client.audio.translations.create` | `translation = client.audio.translations.create(file=f, model="whisper-1")` | Transcribe audio in any language and translate the result to English |

---

## Files

```python
with open("training.jsonl", "rb") as f:
    file = client.files.create(file=f, purpose="fine-tune")
print(file.id)
# Upload a file for fine-tuning

file = client.files.retrieve("file-abc123")
print(file.status)
# Get metadata and processing status of a file

content = client.files.content("file-abc123")
print(content.text)
# Download the raw content of an uploaded file

client.files.delete("file-abc123")
# Delete an uploaded file

files = client.files.list()
for f in files:
    print(f.id, f.filename)
# List all uploaded files

file = client.files.wait_for_processing("file-abc123")
# Poll until a file finishes server-side processing
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.files.create` | `file = client.files.create(file=open("data.jsonl","rb"), purpose="fine-tune")` | Upload a file — `purpose` is `"fine-tune"`, `"batch"`, `"assistants"`, etc. |
| `client.files.retrieve` | `file = client.files.retrieve("file-abc123")` | Get metadata including `filename`, `size`, `status`, and `created_at` |
| `client.files.content` | `content = client.files.content("file-abc123")` | Download the raw bytes of an uploaded file |
| `client.files.delete` | `client.files.delete("file-abc123")` | Permanently remove an uploaded file from your organization |
| `client.files.list` | `files = client.files.list()` | Return a paginated list of all uploaded files |
| `client.files.wait_for_processing` | `file = client.files.wait_for_processing("file-abc123")` | Block and poll until server-side file processing completes |

---

## Fine-tuning

```python
job = client.fine_tuning.jobs.create(
    training_file="file-abc123",
    model="gpt-4o-mini"
)
print(job.id)
# Start a fine-tuning job

job = client.fine_tuning.jobs.retrieve("ftjob-abc123")
print(job.status)
# Get the status and details of a fine-tuning job

client.fine_tuning.jobs.cancel("ftjob-abc123")
# Cancel a fine-tuning job in progress

jobs = client.fine_tuning.jobs.list(limit=10)
for j in jobs:
    print(j.id, j.status)
# List all fine-tuning jobs

events = client.fine_tuning.jobs.list_events("ftjob-abc123")
for e in events:
    print(e.message)
# Stream log events from a fine-tuning job

checkpoints = client.fine_tuning.jobs.checkpoints.list("ftjob-abc123")
for c in checkpoints:
    print(c.fine_tuned_model_checkpoint)
# List intermediate checkpoints saved during fine-tuning
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.fine_tuning.jobs.create` | `job = client.fine_tuning.jobs.create(training_file="file-abc123", model="gpt-4o-mini")` | Start a fine-tuning job on a base model using an uploaded JSONL training file |
| `client.fine_tuning.jobs.retrieve` | `job = client.fine_tuning.jobs.retrieve("ftjob-abc123")` | Check the status (`queued`, `running`, `succeeded`, `failed`) of a job |
| `client.fine_tuning.jobs.cancel` | `client.fine_tuning.jobs.cancel("ftjob-abc123")` | Abort a queued or running fine-tuning job |
| `client.fine_tuning.jobs.list` | `jobs = client.fine_tuning.jobs.list(limit=10)` | List fine-tuning jobs, optionally paginated with `limit` and `after` |
| `client.fine_tuning.jobs.list_events` | `events = client.fine_tuning.jobs.list_events("ftjob-abc123")` | Stream training log events including loss metrics at each step |
| `client.fine_tuning.jobs.checkpoints.list` | `ckpts = client.fine_tuning.jobs.checkpoints.list("ftjob-abc123")` | List saved mid-training checkpoints — each has a usable model ID |

---

## Batch

```python
batch = client.batches.create(
    input_file_id="file-abc123",
    endpoint="/v1/chat/completions",
    completion_window="24h"
)
print(batch.id)
# Submit a JSONL file of requests to run as a batch

batch = client.batches.retrieve("batch_abc123")
print(batch.status)
# Check the status of a batch job

client.batches.cancel("batch_abc123")
# Cancel a batch job that hasn't started yet

batches = client.batches.list()
for b in batches:
    print(b.id, b.status)
# List all batch jobs
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.batches.create` | `batch = client.batches.create(input_file_id="file-abc123", endpoint="/v1/chat/completions", completion_window="24h")` | Submit a JSONL file of requests for async processing at 50% lower cost |
| `client.batches.retrieve` | `batch = client.batches.retrieve("batch_abc123")` | Poll the batch status — `validating`, `in_progress`, `completed`, `failed`, etc. |
| `client.batches.cancel` | `client.batches.cancel("batch_abc123")` | Cancel a batch before it finishes processing |
| `client.batches.list` | `batches = client.batches.list()` | List all batch jobs for your organization |

---

## Moderations

```python
result = client.moderations.create(
    input="Some text to check.",
    model="omni-moderation-latest"
)
print(result.results[0].flagged)
print(result.results[0].categories)
# Check text for policy violations and get category flags

result = client.moderations.create(
    input=[
        {"type": "text", "text": "Check this text."},
        {"type": "image_url", "image_url": {"url": "https://example.com/img.jpg"}}
    ],
    model="omni-moderation-latest"
)
# Moderate both text and image in a single call
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.moderations.create` | `result = client.moderations.create(input="Some text.", model="omni-moderation-latest")` | Check text or images against OpenAI's usage policies — free to use |
| `client.moderations.create` | `result = client.moderations.create(input=[{"type": "text", ...}, {"type": "image_url", ...}], model=...)` | Moderate both text and image inputs together in a single API call |
| `result.results[0].flagged` | `flagged = result.results[0].flagged` | `True` if any policy category was violated; `False` if content is safe |
| `result.results[0].categories` | `cats = result.results[0].categories` | Object with per-category boolean flags (e.g. `hate`, `violence`, `self-harm`) |
| `result.results[0].category_scores` | `scores = result.results[0].category_scores` | Float confidence scores (0–1) per violation category |

---

## Models

```python
models = client.models.list()
for m in models:
    print(m.id)
# List all models available to your API key

model = client.models.retrieve("gpt-4o")
print(model.created, model.owned_by)
# Get details about a specific model

client.models.delete("ft:gpt-4o-mini:org:custom:id")
# Delete a fine-tuned model you own
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.models.list` | `models = client.models.list()` | List all models your API key can access, including fine-tuned models |
| `client.models.retrieve` | `model = client.models.retrieve("gpt-4o")` | Get metadata for a model — `id`, `created`, `owned_by` |
| `client.models.delete` | `client.models.delete("ft:gpt-4o-mini:org:custom:id")` | Permanently delete a fine-tuned model that belongs to your organization |

---

## Error Handling

```python
import openai

try:
    client.chat.completions.create(model="gpt-4o", messages=[...])
except openai.AuthenticationError as e:
    print("Invalid API key:", e.status_code)
except openai.RateLimitError as e:
    print("Rate limited — back off:", e.status_code)
except openai.BadRequestError as e:
    print("Bad request:", e.body)
except openai.APIConnectionError:
    print("Network error — check connection")
except openai.APIStatusError as e:
    print(f"HTTP {e.status_code}: {e.response}")
# Catch all major error types

import time

for attempt in range(3):
    try:
        response = client.chat.completions.create(...)
        break
    except openai.RateLimitError:
        time.sleep(2 ** attempt)
# Simple exponential backoff on rate limit
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `openai.AuthenticationError` | `except openai.AuthenticationError as e:` | Raised on HTTP 401 — invalid, missing, or revoked API key |
| `openai.RateLimitError` | `except openai.RateLimitError as e:` | Raised on HTTP 429 — too many requests; implement exponential backoff |
| `openai.BadRequestError` | `except openai.BadRequestError as e:` | Raised on HTTP 400 — malformed parameters or invalid input content |
| `openai.APIConnectionError` | `except openai.APIConnectionError:` | Raised when the SDK cannot reach the API — network or DNS failure |
| `openai.APIStatusError` | `except openai.APIStatusError as e:` | Base class for all non-2xx HTTP errors; exposes `.status_code` and `.response` |
| `e.status_code` | `print(e.status_code)` | The HTTP status code returned by the server |
| `e.body` | `print(e.body)` | Parsed response body containing OpenAI's error message detail |

---

## Pagination

```python
page = client.models.list()
for model in page:
    print(model.id)
# Auto-paginate — SDK fetches next pages automatically

while True:
    page = client.fine_tuning.jobs.list(after=cursor, limit=20)
    for job in page.data:
        print(job.id)
    if not page.has_next_page():
        break
    cursor = page.data[-1].id
# Manual pagination with cursor
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.*.list` | `for item in client.models.list():` | Auto-pagination — the SDK transparently fetches subsequent pages |
| `page.data` | `items = page.data` | List of resource objects on the current page |
| `page.has_next_page` | `if not page.has_next_page(): break` | Returns `True` if more pages are available |
| `page.next_page` | `page = page.next_page()` | Manually fetch and return the next page object |
| `after` | `client.fine_tuning.jobs.list(after=cursor, limit=20)` | Cursor-based pagination — pass the last item's ID as `after` |

---

## Realtime API (WebSocket)

```python
import asyncio
from openai import AsyncOpenAI

client = AsyncOpenAI()

async def main():
    async with client.beta.realtime.connect(model="gpt-4o-realtime-preview") as conn:
        await conn.session.update(session={"modalities": ["text"]})
        await conn.conversation.item.create(item={
            "type": "message",
            "role": "user",
            "content": [{"type": "input_text", "text": "Say hello!"}]
        })
        await conn.response.create()
        async for event in conn:
            if event.type == "response.text.delta":
                print(event.delta, end="", flush=True)
            elif event.type == "response.done":
                break

asyncio.run(main())
# Full real-time text conversation over WebSocket
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `client.beta.realtime.connect` | `async with client.beta.realtime.connect(model="gpt-4o-realtime-preview") as conn:` | Open a persistent WebSocket session for real-time audio or text exchange |
| `conn.session.update` | `await conn.session.update(session={"modalities": ["text"]})` | Reconfigure the session mid-call — set voice, modalities, tools, turn detection |
| `conn.conversation.item.create` | `await conn.conversation.item.create(item={"type": "message", "role": "user", ...})` | Inject a message or function result into the live conversation context |
| `conn.response.create` | `await conn.response.create()` | Prompt the model to generate a response on the current conversation state |
| `async for event in conn` | `async for event in conn: print(event.type)` | Iterate over server-sent events — text deltas, audio bytes, function calls |
| `event.type` | `if event.type == "response.text.delta":` | Check the event type to route text deltas, audio, or completion signals |

---

## Utilities

```python
print(response._request_id)
# Server-assigned request ID from x-request-id header — log for support tickets

data = response.model_dump()
# Serialize any response object to a plain Python dict

json_str = response.to_json()
# Serialize a response object to a JSON string

completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    seed=42
)
print(completion.system_fingerprint)
# Deterministic sampling — same seed + fingerprint = reproducible output

print(completion.usage.prompt_tokens)
print(completion.usage.completion_tokens)
print(completion.usage.total_tokens)
# Token usage from any completion response
```

| Class / Function | Usage | Description |
|------------------|-------|-------------|
| `response._request_id` | `print(response._request_id)` | Server-assigned `x-request-id` header value — include in bug reports |
| `response.model_dump` | `data = response.model_dump()` | Serialize any Pydantic response object to a plain Python dict |
| `response.to_json` | `json_str = response.to_json()` | Serialize a response object to a JSON string |
| `seed` | `client.chat.completions.create(..., seed=42)` | Enable deterministic generation — same seed yields the same output |
| `completion.system_fingerprint` | `print(completion.system_fingerprint)` | Identifies the backend config — changes signal non-determinism even with a fixed seed |
| `completion.usage.prompt_tokens` | `print(completion.usage.prompt_tokens)` | Number of tokens consumed by the input messages |
| `completion.usage.completion_tokens` | `print(completion.usage.completion_tokens)` | Number of tokens generated in the model's response |
| `completion.usage.total_tokens` | `print(completion.usage.total_tokens)` | Total tokens used — `prompt_tokens + completion_tokens` |
| `openai.NOT_GIVEN` | `client.chat.completions.create(..., stop=openai.NOT_GIVEN)` | Sentinel to explicitly omit an optional parameter (distinct from `None`) |