# Google Gen AI Python SDK

`pip install google-genai` · Python 3.9+

---

## Table of Contents

- [Client Initialization](#client-initialization)
- [Text Generation / Messages](#text-generation--messages)
- [Streaming](#streaming)
- [Async Usage](#async-usage)
- [Tool / Function Calling](#tool--function-calling)
- [Vision / Multimodal](#vision--multimodal)
- [Structured Outputs](#structured-outputs)
- [Embeddings](#embeddings)
- [Error Handling](#error-handling)
- [Utilities & Metadata](#utilities--metadata)

---

## Client Initialization

Initialize the client for Developer API or Vertex AI mode.

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

## Text Generation / Messages

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

## Streaming

Streams text generation chunks iteratively.

### `client.models.generate_content_stream(...)`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `model` | `str` | *Required* | Targeted model identifier. |
| `contents` | `str \| list` | *Required* | Text blocks or combined asset references. |
| `config` | `types.GenerateContentConfig` | `None` | Generation parameters block object. |

```python
from google import genai

client = genai.Client()

# 1. Initialize text generation chunk stream
response_stream = client.models.generate_content_stream(
    model="gemini-2.5-flash",
    contents="Tell a lengthy story describing the lifecycle of an apple tree."
)

# 2. Loop continuously over iterative chunks returned from network
for chunk in response_stream:
    print(chunk.text, end="", flush=True)
print()
```

---

## Async Usage

Use the native asynchronous client sub-layer `.aio`.

```python
import asyncio
from google import genai

# 1. Initialize client and point explicitly to the asynchronous sub-layer
client = genai.Client()
async_client = client.aio

async def main():
    # 2. Execute standard non-blocking invocation
    response = await async_client.models.generate_content(
        model="gemini-2.5-flash",
        contents="Give me an unique name for an indie tech start-up."
    )
    print(response.text)
    
    # 3. Stream chunks over asynchronous loop interfaces
    stream_response = await async_client.models.generate_content_stream(
        model="gemini-2.5-flash",
        contents="Write a 3 paragraph essay about photosynthesis."
    )
    async for chunk in stream_response:
        print(chunk.text, end="", flush=True)

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Tool / Function Calling

Configure executable tools and function references.

### `class types.GenerateContentConfig(tools, tool_config)`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `tools` | `list` | `None` | A structural array of executable tool options or explicit function references. |
| `tool_config` | `types.ToolConfig` | `None` | Configuration parameters determining explicit execution modes. |

```python
from google import genai
from google.genai import types

client = genai.Client()

# 1. Define python target execution functions
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

## Vision / Multimodal

Pass image and asset references directly inside contents array.

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

Define target JSON schemas to enforce structured model outputs.

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

## Embeddings

Convert strings and texts into numeric vector representations.

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

Manage API failures and request exceptions.

### Exception Hierarchy Reference

```
google.genai.errors.APIError
 ├── google.genai.errors.ClientError  (4xx Errors)
 └── google.genai.errors.ServerError  (5xx Errors)
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

## Utilities & Metadata

Inspect execution telemetry and metadata.

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