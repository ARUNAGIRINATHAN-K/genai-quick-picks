# Google Gen AI Python SDK

`pip install google-genai` · Python 3.9+

The unified `google-genai` SDK is the official package for accessing Google's Gemini models across the Gemini Developer API and Google Cloud Vertex AI.

---

## Table of Contents

- [Client Initialization](#client-initialization)
- [Model Management](#model-management)
- [Text Generation](#text-generation)
- [Stateful Chat Sessions](#stateful-chat-sessions)
- [Streaming](#streaming)
- [Async Usage](#async-usage)
- [Files API (Multimodal Uploads)](#files-api-multimodal-uploads)
- [Context Caching](#context-caching)
- [Tool & Function Calling](#tool--function-calling)
- [Google Search Grounding](#google-search-grounding)
- [Vision / Multimodal](#vision--multimodal)
- [Structured Outputs](#structured-outputs)
- [Model Tuning (Fine-Tuning)](#model-tuning-fine-tuning)
- [Image Generation (Imagen)](#image-generation-imagen)
- [Gemini Live API (WebSockets)](#gemini-live-api-websockets)
- [Embeddings](#embeddings)
- [Error Handling](#error-handling)
- [Utilities, Token Counting & Metadata](#utilities-token-counting--metadata)

---

## Client Initialization

Initialize the client for standard Developer API or Google Cloud Vertex AI mode.

### `class google.genai.Client`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `api_key` | `str` | `os.environ.get("GEMINI_API_KEY")` | Your Gemini Developer API Key. |
| `vertexai` | `bool` | `False` | Set to `True` to route requests through Google Cloud Vertex AI instead of Developer API. |
| `project` | `str` | `os.environ.get("GOOGLE_CLOUD_PROJECT")` | Your Google Cloud Project ID (Vertex AI mode only). |
| `location` | `str` | `os.environ.get("GOOGLE_CLOUD_LOCATION")` | Google Cloud target region, e.g., `"us-central1"` (Vertex AI mode only). |
| `http_options` | `types.HttpOptions` | `None` | Configuration object for customizing timeout parameters or API versions. |

```python
import os
from google import genai
from google.genai import types

# 1. Standard Developer API Initialization
client = genai.Client(
    api_key=os.environ.get("GEMINI_API_KEY")
)

# 2. Advanced Vertex AI Mode Initialization
vertex_client = genai.Client(
    vertexai=True,
    project="my-gcp-project-id",
    location="us-central1",
    http_options=types.HttpOptions(api_version="v1")
)

# 3. Accessing the Async Interface
# The async layer is natively available via the `.aio` property attribute
async_client = genai.Client().aio
```

---

## Model Management

Query and manage the metadata of available models.

### `client.models.list()` & `client.models.get(...)`

```python
from google import genai

client = genai.Client()

# 1. List all available models
print("Available Models:")
for model in client.models.list():
    print(f"- {model.name} (Supported actions: {model.supported_generation_methods})")

# 2. Get details for a specific model
model_info = client.models.get(model="gemini-2.5-flash")
print(f"\nModel: {model_info.name}")
print(f"Input Token Limit: {model_info.input_token_limit}")
```

---

## Text Generation

Sends a content generation request to the model.

### `client.models.generate_content(...)`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `model` | `str` | *Required* | Targeted model identifier (e.g., `"gemini-2.5-flash"`). |
| `contents` | `str \| list` | *Required* | The raw instruction query text or combined list elements. |
| `config` | `types.GenerateContentConfig` | `None` | Extra hyperparameter adjustments configurations block. |

```python
from google import genai
from google.genai import types

client = genai.Client()

# 1. Define configuration settings using official type objects
generation_config = types.GenerateContentConfig(
    temperature=0.4,
    top_p=0.95,
    max_output_tokens=1024,
    system_instruction="You are a helpful database administrator."
)

# 2. Execute text content generation call
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Write an optimized SQL query to find second highest salaries.",
    config=generation_config
)

# 3. Direct evaluation parsing of text properties
print(response.text)
```

---

## Stateful Chat Sessions

Create stateful, multi-turn chat sessions where conversation history is automatically maintained by the client.

### `client.chats.create(...)`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `model` | `str` | *Required* | Model identifier for the chat session. |
| `config` | `types.GenerateContentConfig` | `None` | System instructions, safety settings, temperature, and other configuration parameters. |
| `history` | `list` | `None` | Pre-populate the conversation history with list of `types.Content`. |

```python
from google import genai
from google.genai import types

client = genai.Client()

# 1. Create a stateful chat session
chat = client.chats.create(
    model="gemini-2.5-flash",
    config=types.GenerateContentConfig(
        system_instruction="Speak like a medieval knight.",
        temperature=0.7
    )
)

# 2. Send the first message
response1 = chat.send_message("My kingdom is facing a draught. What should I do?")
print(f"Knight: {response1.text}\n")

# 3. Send a follow-up (history is automatically passed)
response2 = chat.send_message("Do you know of any wizards who can help?")
print(f"Knight: {response2.text}\n")

# 4. Access the full message history
for message in chat.get_history():
    print(f"[{message.role}]: {message.parts[0].text}")
```

---

## Streaming

Stream generated content chunks in real-time as they are produced by the model.

### `client.models.generate_content_stream(...)` & `chat.send_message_stream(...)`

```python
from google import genai

client = genai.Client()

# 1. Single-turn streaming
response_stream = client.models.generate_content_stream(
    model="gemini-2.5-flash",
    contents="Tell a lengthy story describing the lifecycle of an apple tree."
)

for chunk in response_stream:
    print(chunk.text, end="", flush=True)
print("\n")

# 2. Chat session streaming
chat = client.chats.create(model="gemini-2.5-flash")
chat_stream = chat.send_message_stream("Write a short poem about coding.")

for chunk in chat_stream:
    print(chunk.text, end="", flush=True)
print()
```

---

## Async Usage

Use the native asynchronous layer of the SDK for concurrent and non-blocking operations.

```python
import asyncio
from google import genai

client = genai.Client()

async def main():
    # 1. Use client.aio context manager to clean up resources automatically
    async with genai.Client().aio as aclient:
        # 2. Execute standard non-blocking invocation
        response = await aclient.models.generate_content(
            model="gemini-2.5-flash",
            contents="Give me an unique name for an indie tech start-up."
        )
        print(f"Company Name: {response.text}\n")
        
        # 3. Stream chunks over asynchronous loop interfaces
        stream_response = await aclient.models.generate_content_stream(
            model="gemini-2.5-flash",
            contents="Write a 3 paragraph essay about photosynthesis."
        )
        async for chunk in stream_response:
            print(chunk.text, end="", flush=True)
        print("\n")

        # 4. Async Chat Streaming
        async_chat = aclient.chats.create(model="gemini-2.5-flash")
        # Await the stream method itself, then iterate using async for
        async_chat_stream = await async_chat.send_message_stream("Explain recursion in one sentence.")
        async for chunk in async_chat_stream:
            print(chunk.text, end="", flush=True)
        print()

if __name__ == "__main__":
    # Ensure correct event loop runner
    asyncio.run(main())
```

---

## Files API (Multimodal Uploads)

Upload large assets (documents, images, videos, audio) for processing by models with large context windows.

### `client.files.upload(...)`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `file` | `str \| io.BytesIO` | *Required* | Path to a local file, or a bytes-like file object to upload. |
| `mime_type` | `str` | `None` | Explicitly set the MIME type. Highly recommended for files without extensions. |

```python
from google import genai

client = genai.Client()

# 1. Upload a large video file or document
uploaded_file = client.files.upload(file="large_manual.pdf")
print(f"File uploaded. Resource Name: {uploaded_file.name}")

# 2. Reference the uploaded file directly in generate_content
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[uploaded_file, "Summarize section 4.2 of this manual."]
)
print(response.text)

# 3. List uploaded files
print("\nUploaded Files:")
for file in client.files.list():
    print(f"- {file.display_name} (Name: {file.name})")

# 4. Clean up / delete the file from Google servers
client.files.delete(name=uploaded_file.name)
print(f"Deleted file: {uploaded_file.name}")
```

---

## Context Caching

Cache massive, static datasets (e.g., long videos, entire codebases, large manuals) to significantly reduce latency and token costs for repeated queries.

### `client.caches.create(...)`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `model` | `str` | *Required* | The model you plan to query with this cache. |
| `config` | `types.CreateCachedContentConfig` | *Required* | Configuration specifying `contents`, `ttl` (e.g. `"3600s"`), and optionally `system_instruction` / `tools`. |

```python
from google import genai
from google.genai import types

client = genai.Client()

# 1. Upload a heavy document first
large_doc = client.files.upload(file="corporate_knowledgebase.pdf")

# 2. Create the context cache (must specify model and ttl)
cache = client.caches.create(
    model="gemini-2.5-flash",
    config=types.CreateCachedContentConfig(
        contents=[large_doc],
        ttl="1800s", # Time To Live: 30 minutes
        display_name="corporate_manual_cache"
    )
)
print(f"Cache created. Name: {cache.name}, Expires: {cache.expire_time}")

# 3. Query the model using the cache
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What is the policy on remote work?",
    config=types.GenerateContentConfig(
        cached_content=cache.name
    )
)
print(response.text)

# 4. List all caches and clean up by deleting
for c in client.caches.list():
    print(f"Cache ID: {c.name}")

client.caches.delete(name=cache.name)
```

---

## Tool & Function Calling

Provide python functions to the model as executable tools. The model can automatically declare which function to invoke and parse the parameters.

### `class types.GenerateContentConfig(tools, tool_config)`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `tools` | `list` | `None` | A structural array of executable tool options or explicit function references. |
| `tool_config` | `types.ToolConfig` | `None` | Configuration parameters determining explicit execution modes. |

```python
from google import genai
from google.genai import types

client = genai.Client()

# 1. Define python target execution functions with type hints and docstrings
def calculate_shipping_cost(destination: str, weight: float) -> float:
    """Calculate target logistics shipping costs based on weights and cities.

    Args:
        destination: Targeted delivery city.
        weight: Package heavy metric values.
    """
    if "New York" in destination:
        return weight * 2.5
    return weight * 1.5

# 2. Bind raw functions directly inside config tool declarations
config = types.GenerateContentConfig(
    tools=[calculate_shipping_cost]
)

# 3. Send prompt trigger statement to the model
messages = ["What is the total cost to ship a 10lb parcel to New York?"]
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=messages,
    config=config
)

# 4. Check for active function execution proposals returned by model
if response.function_calls:
    for call in response.function_calls:
        # Simulate local execution routing inside execution handlers
        if call.name == "calculate_shipping_cost":
            result = calculate_shipping_cost(**call.args)
            
            # 5. Return final outcome details back to context thread
            final_response = client.models.generate_content(
                model="gemini-2.5-flash",
                contents=[
                    types.Content(role="user", parts=[types.Part.from_text(text=messages[0])]),
                    response.candidates[0].content, # Assistant function tool request statement
                    types.Content(role="user", parts=[
                        types.Part.from_function_response(
                            name=call.name,
                            response={"result": result}
                        )
                    ])
                ]
            )
            print(final_response.text)
```

---

## Google Search Grounding

Ground the model's responses in Google Search. The model will search Google and retrieve up-to-date information, providing search grounding metadata.

```python
from google import genai
from google.genai import types

client = genai.Client()

# 1. Configure the google_search tool
grounding_tool = types.Tool(
    google_search=types.GoogleSearch()
)

config = types.GenerateContentConfig(
    tools=[grounding_tool],
    temperature=0.0 # Set low temperature for factuality
)

# 2. Call the model
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Who won the UEFA Champions League final in 2026?",
    config=config
)

print(response.text)

# 3. Inspect grounding metadata and search suggestions
if response.candidates[0].grounding_metadata:
    metadata = response.candidates[0].grounding_metadata
    print("\nGrounding Sources:")
    for chunk in metadata.grounding_chunks:
        if chunk.web:
            print(f"- {chunk.web.title}: {chunk.web.uri}")
```

---

## Vision / Multimodal

Submit combination arrays of images, videos, audio, and text directly to the model.

```python
from PIL import Image
from google import genai

client = genai.Client()

# 1. Load native asset objects into execution instances
img = Image.open("sample_blueprint.jpg")

# 2. Pass image references and instruction strings directly as an inline array
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        img,
        "Analyze this technical structure layout diagram and identify points of safety concerns."
    ]
)
print(response.text)
```

---

## Structured Outputs

Enforce output validation schemas to guarantee response structures.

### `class types.GenerateContentConfig(response_mime_type, response_schema)`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `response_mime_type` | `str` | `None` | Set format type targets, specifically `"application/json"`. |
| `response_schema` | `type \| dict` | `None` | Target blueprint layout parsing definitions (Pydantic model classes or JSON schemas). |

```python
import json
from pydantic import BaseModel, Field
from google import genai
from google.genai import types

client = genai.Client()

# 1. Create a structured target verification framework schema using Pydantic
class MovieReviewExtraction(BaseModel):
    movie_title: str = Field(description="The formal title name of the film production.")
    score: int = Field(description="Rating scoring parameters out of 100 points.")
    sentiment: str = Field(description="Single token emotional markers, e.g. positive, mixed, negative.")

# 2. Instruct the configuration parameters block to enforce schema models
config = types.GenerateContentConfig(
    response_mime_type="application/json",
    response_schema=MovieReviewExtraction,
    temperature=0.1
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="The latest blockbuster 'Nebula Journey' was a spectacular failure. 20/100 completely unwatchable.",
    config=config
)

# 3. Clean validation verification on generated text payloads
structured_json = json.loads(response.text)
print(structured_json["movie_title"], structured_json["score"])
```

---

## Model Tuning (Fine-Tuning)

Initiate and monitor supervised fine-tuning jobs on Gemini base models.

### `client.tunings.create(...)`

```python
from google import genai
from google.genai import types

client = genai.Client()

# 1. Launch a fine-tuning job
# Data must contain supervised conversation examples (e.g. types.TuningDataset)
tuning_job = client.tunings.create(
    model="models/gemini-1.5-flash-001",
    training_data=types.TuningDataset(
        gcs_uri="gs://my-bucket-name/training_data.jsonl"
    ),
    config=types.CreateTuningJobConfig(
        epoch_count=5,
        batch_size=4,
        learning_rate=0.001
    )
)
print(f"Tuning job started. ID: {tuning_job.name}, Status: {tuning_job.state}")

# 2. Monitor or get details on a tuning job
status = client.tunings.get(name=tuning_job.name)
print(f"Current state: {status.state}")

# 3. Use the tuned model once the state is FINISHED
if status.state == "FINISHED":
    response = client.models.generate_content(
        model=status.tuned_model.model, # Reference the tuned model path
        contents="Predict next token based on training style."
    )
    print(response.text)
```

---

## Image Generation (Imagen)

Generate images programmatically using Google's Imagen model.

### `client.models.generate_images(...)`

```python
from google import genai
from google.genai import types

client = genai.Client()

# 1. Generate an image from a prompt
response = client.models.generate_images(
    model="imagen-3.0-generate-002",
    prompt="A futuristic city with flying cars at sunset, high detailed, 4k",
    config=types.GenerateImagesConfig(
        number_of_images=1,
        aspect_ratio="16:9",
        output_mime_type="image/jpeg"
    )
)

# 2. Save or show the generated image
for idx, image in enumerate(response.generated_images):
    # Image bytes are accessible via the image.image property
    image.image.show()
    image.image.save(f"generated_output_{idx}.jpg")
```

---

## Gemini Live API (WebSockets)

Access low-latency, real-time, bidirectional audio, video, and text interactions over WebSockets using the async client wrapper.

### `client.aio.live.connect(...)`

```python
import asyncio
from google import genai

client = genai.Client()

async def run_live_agent():
    # 1. Establish the connection session context
    # Use the live preview model that supports real-time WebSocket interactions
    model = "gemini-3.1-flash-live-preview"
    config = {"response_modalities": ["AUDIO"]}

    async with client.aio.live.connect(model=model, config=config) as session:
        print("Live Session connected successfully. Start talking...")
        
        # Send text input to the model
        await session.send(input={"text": "Hello, Gemini! Can you hear me?"})
        
        # Listen for real-time responses from the server
        async for response in session.receive():
            server_content = response.server_content
            if server_content and server_content.model_turn:
                for part in server_content.model_turn.parts:
                    # Capture text chunk or raw audio bytes
                    if part.text:
                        print(part.text, end="", flush=True)
                    elif part.inline_data:
                        # part.inline_data.data contains the PCM raw audio payload
                        pass

if __name__ == "__main__":
    asyncio.run(run_live_agent())
```

---

## Embeddings

Convert string inputs into numeric vector representations for downstream tasks (similarity search, classification, clustering).

### `client.models.embed_content(...)`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `model` | `str` | *Required* | Vector conversion model selection targets (e.g., `"text-embedding-004"`). |
| `contents` | `str \| list` | *Required* | High-volume string text targets for vector transformation conversion operations. |
| `config` | `types.EmbedContentConfig` | `None` | Optional configurations block parameters. |

```python
from google import genai

client = genai.Client()

# 1. Convert textual strings over to numeric vector payloads
response = client.models.embed_content(
    model="text-embedding-004",
    contents="Machine Learning platforms deployment processes documentation details."
)

# 2. Extract dimensions array outputs from structural objects elements
vector_values = response.embeddings[0].values
print(f"Generated Vector Dimensions Length Count: {len(vector_values)}")
```

---

## Error Handling

Handle standard errors, rate limits, and network issues gracefully using the client exception hierarchy.

### Exception Hierarchy Reference
```
google.genai.errors.APIError
 ├── google.genai.errors.ClientError  (4xx Errors, e.g. 400, 401, 403, 404, 429)
 └── google.genai.errors.ServerError  (5xx Errors, e.g. 500, 503)
```

```python
from google import genai
from google.genai import errors

client = genai.Client()

try:
    # Attempt operation sequence
    response = client.models.generate_content(
        model="gemini-invalid-model-name-string",
        contents="Hello World"
    )
except errors.ClientError as e:
    # Catches 400, 401, 403, 404, 429 errors
    print(f"Handled invalid configuration requests adjustments errors: {e.message} (Code: {e.code})")
except errors.ServerError as e:
    # Catches 500, 503 infrastructure glitches
    print(f"Google production environment processing error detected: {e.message}")
except errors.APIError as e:
    # Base fallback catch-all
    print(f"A generic API failure has occurred: {e}")
```

---

## Utilities, Token Counting & Metadata

Query token statistics and inspect complete telemetry logs.

### `client.models.count_tokens(...)`

```python
from google import genai

client = genai.Client()

# 1. Calculate input token volume before sending queries
token_info = client.models.count_tokens(
    model="gemini-2.5-flash",
    contents="What are the basic ingredients of a bread dough recipe?"
)
print(f"Token Count: {token_info.total_tokens}")
```

### Response Objects Field Map Reference

| Property Attribute | Type | Description |
|---|---|---|
| `text` | `str` | High-level quick accessor for parsing output text parts directly. |
| `candidates` | `list` | Response alternatives returned through completion pipeline frameworks. |
| `usage_metadata` | `types.UsageMetadata` | Structural fields tracking context sizing tokens telemetry. |
| `function_calls` | `list` | Direct collection parsing function execution blocks. |

```python
from google import genai

client = genai.Client()

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What are the basic ingredients of a bread dough recipe?"
)

# 1. Fetch system evaluation metric properties
metadata = response.usage_metadata
print(f"Input Prompt Token Volume Count: {metadata.prompt_token_count}")
print(f"Output Generation Token Volume Count: {metadata.candidates_token_count}")
print(f"Combined Total Structural Usage: {metadata.total_token_count}")
```