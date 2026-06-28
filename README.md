[![skills.sh](https://skills.sh/b/mdapiio/mdapi.io)](https://skills.sh/mdapiio/mdapi.io)

# mdapi.io — Minimal Data API I/O: a content transformation layer primitive for AI systems.


Transforms documents, images, and webpages into AI-ready Markdown and structured data, optimized for LLM efficiency and token usage.

## Agent entrypoint

- **Start AI discovery** → https://mdapi.io/.well-known/ai-discovery.json
- **Use skill** → https://mdapi.io/.well-known/skill.md

## Quick Start

Choose your entry point based on your role:

| Role                                                     | Protocol                     | Endpoint                  | When to use                                                                                              |
|----------------------------------------------------------|------------------------------|---------------------------|----------------------------------------------------------------------------------------------------------|
| IDE / coding agent (JetBrains, Cursor, VS Code, etc.)    | ACP (Agent Client Protocol)  | POST /acp                 | You are an IDE plugin or coding agent. Use tools/call with convert tool.                                 |
| AI agent (Claude Code, Codex, OpenClaw, Hermes, etc.)    | A2A (Agent-to-Agent)         | POST /a2a                 | You are an autonomous agent. Use SendMessage with text/file parts. Supports streaming and task tracking. |
| AI agent (any framework)                                 | MCP (Model Context Protocol) | GET  /mcp + POST /mcp     | You need tool discovery. Use tools/call with convert tool.                                               |
| OpenAI-compatible client                                 | OpenAI API                   | POST /v1/chat/completions | You already use OpenAI SDK. Pass URL/file in messages. Supports streaming.                               |
| Direct HTTP / curl / script                              | REST API                     | GET  / or POST /          | Simplest path. GET returns Markdown directly. POST returns JSON with metadata.                           |

### Universal discovery

All protocols and capabilities are described in one file:
GET /.well-known/ai-discovery.json

## Features

- Stateless, in-memory processing
- Edge execution with automatic scaling
- Prompt-driven transformation
- AI‑optimized output for LLMs
- Pay-per-use via x402 v1/v2 or manual payment

## Supported Formats

| Type      | Formats                        |
|-----------|--------------------------------|
| Documents | PDF, DOCX, XLSX, XLS, ODT, ODS |
| Images    | JPEG, JPG, PNG, WebP, SVG      |
| Text      | HTML, XML, JSON, CSV, TXT      |
| Webpages  | Any publicly accessible URL    |

## Limits

| Limit               | Value                                     |
|---------------------|-------------------------------------------|
| **Max file size**   | 50 MB                                     |
| **Max URL content** | 50 MB                                     |
| **Rate limit**      | 10,000 requests per hour                  |
| **Free tier**       | 10 requests per day (no token required)   |
| **Paid tier**       | min $0.01 per conversion (USDC on Solana) |
| **Token validity**  | 1 year                                    |

## Authentication

**Recommended:** Use `Authorization: Bearer TOKEN`

| Method               | Use Case                                                 |
|----------------------|----------------------------------------------------------|
| Bearer (recommended) | `-H "Authorization: Bearer TOKEN"`                       |
| Header (x402)        | `-H "X-Token-Required: TOKEN"` (for x402 legacy clients) |
| Query (legacy)       | `?token=TOKEN` (for simple GET requests)                 |

## Token Activation

Before you can use a paid token, you must receive a 402 response first:

| Step | Description                                             |
|------|---------------------------------------------------------|
| 1    | Request without token → Receive 402 with NEW token+memo |
| 2    | Send USDC on Solana to wallet with memo from 402        |
| 3    | Retry with EXACT token+memo from 402 → Activation       |
| 4    | After: use token only (no memo needed)                  |

Important: The token+memo issued in the 402 response MUST be used exactly. Using old token or different memo will be rejected.

## API Usage

### GET / (URL and text conversion)

Simple URL or text conversion using query parameters. Returns Markdown directly.

#### Parameters

| Parameter | Type    | Required | Description                                      |
|-----------|---------|----------|--------------------------------------------------|
| `url`     | string  | *        | URL to convert to Markdown                       |
| `text`    | string  | *        | Direct text content to process                   |
| `prompt`  | string  |          | Custom instructions for LLM processing           |
| `result`  | string  |          | Response format: `markdown`, `prompt`, or `both` |
| `stream`  | boolean |          | Enable streaming: `true` for SSE response        |
| `token`   | string  |          | Access token for paid tier                       |
| `memo`    | string  |          | Memo for token activation                        |

*One of `url` or `text` is required.*

### POST / (URL, text, or file upload)

Supports URL conversion, direct text processing, and multipart file uploads. Returns a JSON object containing the Markdown content.

#### Parameters

| Parameter | Type    | Required | Description                                      |
|-----------|---------|----------|--------------------------------------------------|
| `url`     | string  | *        | URL to convert to Markdown                       |
| `file`    | file    | *        | File to upload (multipart)                       |
| `text`    | string  | *        | Direct text content to process                   |
| `prompt`  | string  |          | Custom instructions for LLM processing           |
| `result`  | string  |          | Response format: `markdown`, `prompt`, or `both` |
| `stream`  | boolean |          | Enable streaming: `true` for SSE response        |
| `token`   | string  |          | Access token for paid tier                       |
| `memo`    | string  |          | Memo for token activation                        |

*One of `url`, `file`, or `text` is required.*

### Result Format Parameter

The `result` parameter controls the response format for both GET and POST requests.

| Value                | Description                                                     |
|----------------------|-----------------------------------------------------------------|
| `markdown` (default) | Returns the converted Markdown content                          |
| `prompt`             | Returns the result of LLM processing with `prompt` instructions |
| `both`               | Returns both `markdown` and `prompt_result` in the response     |

When `result=both`:

- **GET requests** return Markdown combining `markdown`, followed by "## Prompt Result" and `prompt_result` (always in Markdown format)
- **POST requests** return JSON with `markdown` and `prompt_result` fields

### Prompt Parameter

The `prompt` parameter lets you specify custom instructions for the LLM to follow when generating the result.

| Use Case           | Example                                                 |
|--------------------|---------------------------------------------------------|
| Summarize document | `?url=...&prompt=Summarize this document&result=prompt` |
| Extract key points | `?text=...&prompt=Extract key points&result=prompt`     |
| Convert to JSON    | `?url=...&prompt=Convert to JSON format&result=prompt`  |
| Analyze content    | `?text=...&prompt=Analyze and explain&result=prompt`    |

### Streaming Parameter

The `stream` parameter enables Server-Sent Events (SSE) streaming for real-time response delivery.

| Value  | Description                      |
|--------|----------------------------------|
| `true` | Returns SSE stream with chunks   |

Example:
```bash
curl "https://mdapi.io/?url=...&stream=true"
```

Response format:
```json
{"type":"token_info","status":"valid","balance":99,"expires":2027-01-01}
{"content": " chunk", "resource": "https://example.com/file.pdf", "mimetype": "application/pdf"}
{"content": " more"}
// Resource types: URL, "text", or filename.pdf
data: [DONE]
```

### Response Codes

| Code | Description       | Response Body                              |
|------|-------------------|--------------------------------------------|
| 200  | Success (GET)     | Markdown content                           |
| 200  | Success (POST)    | JSON with markdown and prompt_result       |
| 402  | Payment Required  | x402 payment instructions or requirements  |
| 400  | Bad Request       | `{"error": "Invalid request"}`             |
| 401  | Invalid Token     | `{"error": "Invalid token"}`               |
| 404  | Not Found         | `{"error": "Resource not found"}`          |
| 413  | Payload Too Large | `{"error": "File too large"}`              |
| 429  | Rate Limited      | `{"error": "Rate limit exceeded"}`         |
| 500  | Server Error      | `{"error": "Internal error"}`              |

### Token Status

The X-Token-Status header (and token_status field in responses) indicates the current state of authentication:

| Status             | Description                                    |
|--------------------|------------------------------------------------|
| free               | Free tier (no token required, 10 requests/day) |
| valid              | Paid token active with remaining balance       |
| invalid            | Token not found or not provided                |
| expired            | Token validity period has ended                |
| exhausted          | Token balance has been fully used              |
| expired_pending    | Activation memo has expired                    |
| activated          | Token was just activated with this request     |
| verification_error | Payment verification failed                    |
| invalid_payment    | Payment transaction is invalid                 |
| error              | Internal error during token processing         |
| pending            | Payment required (token not yet activated)     |

### Endpoints

| Method | Path                                                 | Description                                                                                   |
|--------|------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| GET    | /                                                    | API docs or URL/text conversion (via query parameters: `url`, `text`, `prompt`, `result`)     |
| POST   | /                                                    | Convert URL, text, or upload file (supports `url`, `file`, `text`, `prompt`, `result` params) |
| POST   | /v1/chat/completions                                 | OpenAI-compatible endpoint                                                                    |
| GET    | /mcp                                                 | MCP server manifest                                                                           |
| POST   | /mcp                                                 | MCP tool calls                                                                                |
| POST   | /acp                                                 | ACP RPC endpoint (IDE agents)                                                                 |
| POST   | /a2a                                                 | A2A RPC endpoint (agent2agent)                                                                |
| GET    | /health                                              | Health check                                                                                  |
| GET    | /llms.txt                                            | API documentation                                                                             |
| GET    | /llms-full.txt                                       | Full API documentation                                                                        |
| GET    | /.well-known/ai-discovery.json or /ai-discovery.json | AI discovery                                                                                  |
| GET    | /.well-known/agent.json        or /agent.json        | AI Agent discovery                                                                            |
| GET    | /.well-known/agent-card.json   or /agent-card.json   | A2A Agent card                                                                                |
| GET    | /.well-known/acp.json          or /acp.json          | ACP manifest                                                                                  |
| GET    | /.well-known/x402.json         or /x402.json         | x402 payment manifest                                                                         |
| GET    | /.well-known/openapi.json      or /openapi.json      | OpenAPI specification (JSON)                                                                  |
| GET    | /.well-known/openapi.yaml      or /openapi.yaml      | OpenAPI specification (YAML)                                                                  |
| GET    | /.well-known/mapi.md           or /mapi.md           | MAPI specification (case-insensitive path MAPI.md support)                                    |
| GET    | /.well-known/skill.md          or /skill.md          | Skill specification (case-insensitive path SKILL.md support)                                  |

#### Examples

```bash
# URL conversion via GET (free)
curl "https://mdapi.io/?url=https://example.com/file.pdf"

# URL with prompt and result=both (returns markdown + prompt_result)
curl "https://mdapi.io/?url=https://example.com/file.pdf&prompt=Summarize&result=both"

# Text with prompt and result=prompt (return prompt_result)
curl "https://mdapi.io/?text=Your+text+here&prompt=Summarize+this&result=prompt"

# Token activation via GET (activate and use)
curl "https://mdapi.io/?url=https://example.com/file.pdf&token=YOUR_TOKEN&memo=YOUR_MEMO"

# Paid request with token via GET (using token)
curl "https://mdapi.io/?url=https://example.com/file.pdf&token=YOUR_TOKEN"

# URL conversion via POST (free)
curl -X POST -F "url=https://example.com/file.pdf" "https://mdapi.io/"

# File upload (multipart/form-data)
curl -X POST -F "file=@document.pdf" "https://mdapi.io/"

# Token activation via POST (activate and use)
curl -X POST -H "Authorization: Bearer YOUR_TOKEN" -H "X-Memo-Required: YOUR_MEMO" -F "url=https://example.com/file.pdf" "https://mdapi.io/"
curl -X POST -H "X-Token-Required: YOUR_TOKEN" -H "X-Memo-Required: YOUR_MEMO" -F "url=https://example.com/file.pdf" "https://mdapi.io/"
curl -X POST -F "token=YOUR_TOKEN" -F "memo=YOUR_MEMO" -F "url=https://example.com/file.pdf" "https://mdapi.io/"

# Paid request with token via POST (using token)
curl -X POST -H "Authorization: Bearer YOUR_TOKEN" -F "file=@document.pdf" "https://mdapi.io/"
curl -X POST -H "Authorization: Bearer YOUR_TOKEN" -F "url=https://example.com/file.pdf" "https://mdapi.io/"
curl -X POST -H "X-Token-Required: YOUR_TOKEN" -F "url=https://example.com/file.pdf" "https://mdapi.io/"
curl -X POST -F "token=YOUR_TOKEN" -F "url=https://example.com/file.pdf" "https://mdapi.io/"

```

### OpenAI Compatible Endpoint

The `/v1/chat/completions` endpoint provides an OpenAI‑compatible API for markdown conversion with streaming support.

**Supported features:**
- URL extraction from message content (any text containing https?://)
- image_url in messages (OpenAI format) — supports HTTP URLs and data URLs
- file in messages (OpenAI format) — base64 encoded files
- Token and memo via headers (recommended for POST)
- Streaming SSE responses (`stream: true`)
- Custom instructions with LLM processing (system messages and structured JSON extraction, e.g., entities and fields)

#### Request Schema

```json
{
  "type": "object",
  "properties": {
    "model": {
      "type": "string",
      "description": "Model identifier (any string accepted)"
    },
    "messages": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "role": {"type": "string", "enum": ["user", "system", "assistant"]},
          "content": {"oneOf": [{"type": "string"}, {"type": "array"}]}
        }
      },
      "description": "Chat messages. URL in content, image_url or file in content for input"
    },
    "stream": {
      "type": "boolean",
      "default": false,
      "description": "Enable streaming SSE responses"
    }
  },
  "required": ["model", "messages"]
}
```

## MCP Configuration

Connect mdapi.io to your MCP-compatible client.

### Basic Configuration

Add to your MCP config file:

```json
{
  "mcpServers": {
    "mdapi": {
      "url": "https://mdapi.io/mcp"
    }
  }
}
```

> **Note:** No token is required to connect. A free tier is available (10 requests per day).

### OpenClaw Integration

OpenClaw can use mdapi.io in two ways:

**Option 1: Via MCP (Recommended)**
```json
{
  "mcpServers": {
    "mdapi": {
      "url": "https://mdapi.io/mcp"
    }
  }
}
```

**Option 2: Via OpenAI-compatible endpoint**
```bash
openclaw config set llm.apiBase https://mdapi.io/v1
openclaw config set llm.apiKey YOUR_TOKEN
```

### Using MCP with a token

For paid tier, include token in configuration:

```json
{
  "mcpServers": {
    "mdapi": {
      "url": "https://mdapi.io/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      }
    }
   }
}
```

### Token activation via MCP

To activate a new token, include memo in the first request:

```json
{
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "url": "https://example.com/doc.pdf",
      "token": "YOUR_NEW_TOKEN",
      "memo": "YOUR_PAYMENT_MEMO"
    }
  }
}
```

After activation, use token only (no memo needed):

```json
{
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "url": "https://example.com/doc.pdf",
      "token": "YOUR_ACTIVATED_TOKEN"
    }
  }
}
```

### MCP Tool Examples

Convert with prompt and result:

```json
{
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "url": "https://example.com/doc.pdf",
      "prompt": "Summarize this document",
      "result": "both",
      "token": "YOUR_TOKEN"
    }
  }
}
```

Process text directly:

```json
{
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "text": "Your text content here",
      "prompt": "Extract key points",
      "result": "prompt"
    }
  }
}
```

### Using Environment Variables

Or use environment variable:

```bash
export MDAPI_TOKEN=YOUR_ACTIVATED_TOKEN
```

## Code Examples

### JavaScript (fetch)

```javascript
// Convert a URL via GET
const url = 'https://mdapi.io/?url=https://example.com/doc.pdf';
const response = await fetch(url);

if (!response.ok) {
  throw new Error(`HTTP ${response.status}`);
}

const markdown = await response.text();
console.log(markdown);
```

```javascript
// Convert a URL via POST
const formData = new FormData();
formData.append('url', 'https://example.com/doc.pdf');

const response = await fetch('https://mdapi.io/', {
  method: 'POST',
  body: formData
});

if (!response.ok) {
  throw new Error(`HTTP ${response.status}`);
}

const data = await response.json();
console.log(data);
```

```javascript
// Text with prompt
const response = await fetch('https://mdapi.io/?text=Your+text&prompt=Summarize&result=both');

if (!response.ok) {
  throw new Error(`HTTP ${response.status}`);
}

const data = await response.json();
console.log(data.markdown);
console.log(data.prompt_result);
```

### Python

```python
import requests

# Convert a URL via GET
try:
    response = requests.get('https://mdapi.io/?url=https://example.com/doc.pdf')
    response.raise_for_status()  # Raises an HTTPError for bad status
    markdown = response.text
    print(markdown)
except requests.exceptions.RequestException as e:
    print("HTTP error:", e)
```

```python
import requests

# Convert a URL via POST
try:
    response = requests.post(
        'https://mdapi.io/',
        data={'url': 'https://example.com/doc.pdf'}
    )
    response.raise_for_status()
    data = response.json()
    print(data)
except requests.exceptions.RequestException as e:
    print("HTTP error:", e)
```

```python
import requests

# Text with prompt
try:
    response = requests.post(
        'https://mdapi.io/',
        data={'text': 'Your text', 'prompt': 'Summarize', 'result': 'both'}
    )
    response.raise_for_status()
    data = response.json()
    print(data['markdown'])
    print(data['prompt_result'])
except requests.exceptions.RequestException as e:
    print("HTTP error:", e)
```

```python
import requests

# File upload
try:
    with open('document.pdf', 'rb') as f:
        response = requests.post('https://mdapi.io/', files={'file': f})
        response.raise_for_status()
        data = response.json()
        print(data)
except requests.exceptions.RequestException as e:
    print("HTTP error:", e)
```

### Go

```go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "net/http"
)

func main() {
    // Convert a URL via GET
    resp, err := http.Get("https://mdapi.io/?url=https://example.com/doc.pdf")
    if err != nil {
        fmt.Println("HTTP error:", err)
        return
    }
    defer resp.Body.Close()

    body, err := io.ReadAll(resp.Body)
    if err != nil {
        fmt.Println("Read error:", err)
        return
    }

    if resp.StatusCode == 200 {
        fmt.Println(string(body)) // Markdown as text
    } else {
        fmt.Printf("HTTP %d\n%s\n", resp.StatusCode, body)
    }
}
```

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "mime/multipart"
    "net/http"
)

func main() {
    body := &bytes.Buffer{}
    writer := multipart.NewWriter(body)

    err := writer.WriteField("url", "https://example.com/doc.pdf")
    if err != nil {
        fmt.Println("Writer error:", err)
        return
    }
    err = writer.Close()
    if err != nil {
        fmt.Println("Writer close error:", err)
        return
    }

    // Convert a URL via POST
    resp, err := http.Post("https://mdapi.io/", writer.FormDataContentType(), body)
    if err != nil {
        fmt.Println("HTTP error:", err)
        return
    }
    defer resp.Body.Close()

    var result map[string]interface{}
    err = json.NewDecoder(resp.Body).Decode(&result)
    if err != nil {
        fmt.Println("JSON decode error:", err)
        return
    }

    fmt.Println(result["markdown"])
}
```

### Rust

```rust
use anyhow::Result;
use reqwest::Client;

#[tokio::main]
async fn main() -> Result<()> {
    let client = Client::new();
    let url = "https://mdapi.io/?url=https://example.com/doc.pdf";

    let response = client.get(url).send().await?;
    response.error_for_status_ref()?;

    let markdown = response.text().await?;
    println!("{}", markdown);

    Ok(())
}
```

```rust
use anyhow::{Result, Context};
use reqwest::Client;
use serde::Deserialize;

#[derive(Deserialize)]
struct ApiResponse {
    markdown: Option<String>,
    prompt_result: Option<String>,
}

#[tokio::main]
async fn main() -> Result<()> {
    let client = Client::new();
    let token = std::env::var("MDAPI_TOKEN").context("MDAPI_TOKEN not set")?;

    // The `result` field must be set (e.g., "prompt" or "both") when using `prompt` otherwise only markdown is returned.
    let form_data = [
        ("url", "https://example.com/doc.pdf"),
        ("prompt", "Extract key points"),
        ("result", "both"),
    ];

    let response = client
        .post("https://mdapi.io/")
        .header("Authorization", format!("Bearer {}", token))
        .form(&form_data)
        .send()
        .await
        .context("Failed to send request to mdapi.io")?;

    response
        .error_for_status_ref()
        .context("HTTP error from mdapi.io")?;

    let data: ApiResponse = response.json().await?;

    if let Some(markdown) = data.markdown {
        println!("Markdown:\n{}", markdown);
    }
    if let Some(prompt_result) = data.prompt_result {
        println!("Prompt result:\n{}", prompt_result);
    }

    Ok(())
}
```

```rust
use anyhow::Result;
use reqwest::Client;
use std::fs::File;

#[tokio::main]
async fn main() -> Result<()> {
    let client = Client::new();
    let token = std::env::var("MDAPI_TOKEN")?;
    let memo = std::env::var("MDAPI_ACTIVATION_MEMO")?; // memo is required only for first activation

    let form = reqwest::multipart::Form::new()
        .file("file", File::open("document.pdf")?)
        .text("token", &token)
        .text("memo", &memo);

    let response = client
        .post("https://mdapi.io/")
        .multipart(form)
        .send()
        .await?;

    response.error_for_status_ref()?;

    let markdown = response.text().await?;
    println!("{}", markdown);

    Ok(())
}
```

```rust
use anyhow::{Result, Context};
use reqwest::Client;
use serde::Deserialize;
use std::collections::HashMap;

#[derive(Deserialize)]
struct ApiResponse {
    markdown: Option<String>,
    prompt_result: Option<String>,
}

pub struct MdApiClient {
    client: Client,
    base_url: String,
    token: Option<String>,
}

impl MdApiClient {
    pub fn new(token: Option<String>) -> Self {
        Self {
            client: Client::new(),
            base_url: "https://mdapi.io".to_string(),
            token,
        }
    }

    pub async fn convert_url(
        &self,
        url: &str,
        prompt: Option<&str>,
    ) -> Result<String> {
        let mut params = HashMap::new();
        params.insert("url", url);
        if let Some(prompt) = prompt {
            params.insert("prompt", prompt);
            params.insert("result", "both"); // required when using prompt, otherwise only markdown is returned
        }

        let mut request = self.client.post(&self.base_url).form(&params);

        if let Some(ref token) = self.token {
            request = request.header("Authorization", format!("Bearer {}", token));
        }

        let response = request
            .send()
            .await
            .context("Failed to send HTTP request")?;

        response
            .error_for_status_ref()
            .context("API returned error status")?;

        response
            .text()
            .await
            .context("Failed to read response body")
    }
}

// Example usage in an autonomous AI Agent or infrastructure service
#[tokio::main]
async fn main() -> Result<()> {
    let token = std::env::var("MDAPI_TOKEN").ok();
    let client = MdApiClient::new(token);

    let markdown = client
        .convert_url(
            "https://example.com/doc.pdf",
            Some("Summarize this document"), // when using prompt, result="both" is injected automatically
        )
        .await?;

    println!("{}", markdown);
    Ok(())
}
```

```rust
use anyhow::Result;
use futures_util::stream::StreamExt;
use reqwest::Client;

// Streaming response when using the OpenAI‑compatible endpoint with stream = true
#[tokio::main]
async fn main() -> Result<()> {
    let client = Client::new();
    let token = std::env::var("MDAPI_TOKEN")?;

    let body = serde_json::json!({
        "model": "markdown-v1",
        "messages": [
            {
                "role": "user",
                "content": "Convert https://example.com/doc.pdf"
            }
        ],
        "stream": true
    });

    let response = client
        .post("https://mdapi.io/v1/chat/completions")
        .header("Authorization", format!("Bearer {}", token))
        .json(&body)
        .send()
        .await?;

    response.error_for_status_ref()?;

    let mut stream = response.bytes_stream();
    while let Some(chunk) = stream.next().await {
        let chunk = chunk?;
        let text = String::from_utf8_lossy(&chunk);
        eprint!("{}", text);
    }

    Ok(())
}
```

### OpenAI SDK

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://mdapi.io/v1",
    api_key="YOUR_TOKEN"
)

try:
    response = client.chat.completions.create(
        model="markdown-v1",
        messages=[{"role": "user", "content": "Convert https://example.com/doc.pdf"}]
    )
    print(response.choices[0].message.content)
except Exception as e:
    print("API error:", e)
```

## A2A Configuration

Connect mdapi.io to your A2A-compatible agent (Claude Code, Codex, OpenClaw, Hermes, etc.).

### Basic Configuration

Add to your A2A client configuration:

```json
{
  "agent": {
    "name": "mdapi-client",
    "agentCard": {
      "url": "https://mdapi.io/.well-known/agent-card.json"
    }
  }
}
```

Or use JSON-RPC directly:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "SendMessage",
  "params": {
      "message": {
        "messageId": "msg-uuid-1",
         "parts": [
           { "text": "Convert https://example.com/doc.pdf" }
        ]
      }
  }
}
```

### A2A Methods

| Method                     | Description                                      |
|----------------------------|--------------------------------------------------|
| `SendMessage`              | Send a message to initiate conversion            |
| `SendStreamingMessage`     | Send message with SSE streaming updates          |
| `GetTask`                  | Get task status and results by ID                |
| `ListTasks`                | List tasks with optional filtering               |
| `CancelTask`               | Cancel an in-progress task                       |
| `SubscribeToTask`          | Subscribe to task updates via SSE                |

### A2A Examples

#### SendMessage

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "SendMessage",
    "params": {
      "message": {
        "messageId": "msg-uuid-1",
        "parts": [
          { "text": "Convert https://example.com/doc.pdf" }
        ]
      }
    }
  }'
```

#### SendMessage with file (base64)

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "SendMessage",
    "params": {
      "message": {
        "messageId": "msg-uuid-2",
        "parts": [
          {
            "raw": "JVBERi0xLjQK...",
            "filename": "document.pdf",
            "mediaType": "application/pdf"
          }
        ]
      }
    }
  }'
```

#### SendMessage with structured data

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "SendMessage",
    "params": {
      "message": {
        "messageId": "msg-uuid-3",
        "parts": [
          {
            "data": {
              "url": "https://example.com/doc.pdf",
              "result": "markdown"
            },
            "mediaType": "application/json"
          }
        ]
      }
    }
  }'
```

#### Multi-turn conversation (follow-up)

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 4,
    "method": "SendMessage",
    "params": {
      "contextId": "ctx-uuid-1",
      "message": {
        "messageId": "msg-uuid-4",
        "parts": [
          { "text": "Now convert the tables to JSON" }
        ]
      }
    }
  }'
```

#### GetTask

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 5,
    "method": "GetTask",
    "params": { "id": "task_12345" }
  }'
```

#### ListTasks

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 6,
    "method": "ListTasks",
    "params": { "contextId": "ctx_12345", "pageSize": 10 }
  }'
```

### A2A Message Parts

Messages use the A2A `Part` format (field-name discriminators per spec v1.0.0):

| Type  | Description                        | Fields                                  |
|-------|------------------------------------|-----------------------------------------|
| `text`| Plain text content                 | `text`                                  |
| `data`| Structured JSON data (core params) | `data` (object), `mediaType` (optional) |
| `file`| File reference (base64 or inline)  | `raw` (base64), `filename`, `mediaType` |

**Part → Core Parameter Mapping:**
- `text` Part → `text` param (direct content)
- `data` Part → merged as params (`url`, `prompt`, `result`, `token`, `memo`, etc.)
- `raw`/`url` Part → `file` param (base64 decoded or fetched)

### Message Object

```typescript
interface Message {
  messageId: string;                  // REQUIRED: unique ID (e.g. "msg-uuid")
  contextId?: string;                 // Optional: group related tasks
  taskId?: string;                    // Optional: associate with existing task
  role: "ROLE_USER" | "ROLE_AGENT";   // REQUIRED
  parts: Array<Part>;                 // REQUIRED: at least one part
}
```

### Task Data Model

```typescript
interface Task {
  id: string;                         // "task_<timestamp>_<random>"
  contextId: string;                  // "ctx_<timestamp>_<random>"
  status: {
    state: string;                    // "TASK_STATE_COMPLETED" | "TASK_STATE_FAILED" | "TASK_STATE_WORKING" | "TASK_STATE_CANCELED" | "TASK_STATE_REJECTED"
    timestamp: string;                // ISO 8601
    message?: Message;                // only on failure
  };
  artifacts?: Array<{
    artifactId: string;
    name: string;
    parts: Array<Part>;
  }>;
  history?: Array<Message>;
}
```

### Task States

| State                       | Description                   |
|-----------------------------|-------------------------------|
| `TASK_STATE_COMPLETED`      | Task finished successfully    |
| `TASK_STATE_FAILED`         | Task failed during processing |
| `TASK_STATE_WORKING`        | Task is being processed       |
| `TASK_STATE_INPUT_REQUIRED` | Agent needs more input        |
| `TASK_STATE_CANCELED`       | Task was canceled by client   |
| `TASK_STATE_REJECTED`       | Task was rejected by server   |

### Error Responses

A2A uses JSON-RPC 2.0 error format with A2A-specific error codes:

| Code     | Error                  | Description                                                |
|----------|------------------------|------------------------------------------------------------|
| `-32700` | Parse error            | Invalid JSON payload                                       |
| `-32600` | Invalid Request        | Missing required fields (message.parts, message.messageId) |
| `-32601` | Method not found       | Unknown A2A method                                         |
| `-32001` | Task not found         | Task ID does not exist                                     |
| `-32002` | Task not cancelable    | Task in terminal state                                     |
| `-32003` | Cannot subscribe       | Task in terminal state                                     |
| `-32005` | Unsupported media type | Content-Type not accepted                                  |

**Example error response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32600,
    "message": "Invalid Request",
    "data": [
      {
        "@type": "type.googleapis.com/google.rpc.BadRequest",
        "fieldViolations": [
          {
            "field": "message.messageId",
            "description": "Message messageId is required"
          }
        ]
      }
    ]
  }
}
```

### Streaming

Use `SendStreamingMessage` for real-time SSE updates:

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -H "Accept: text/event-stream"   -d '{
    "jsonrpc": "2.0",
    "id": 5,
    "method": "SendStreamingMessage",
    "params": {
      "message": {
        "messageId": "msg-uuid-5",
         "parts": [
           { "text": "Convert https://example.com/doc.pdf" }
         ]
      }
    }
  }'
```

Response format:
```
data: {"jsonrpc":"2.0","id":5,"result":{"task":{...}}}

data: {"jsonrpc":"2.0","id":5,"result":{"task":{...}}}

data: [DONE]
```

### Subscribe to Task

Subscribe to an existing task for real-time updates:

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -H "Accept: text/event-stream"   -d '{
    "jsonrpc": "2.0",
    "id": 6,
    "method": "SubscribeToTask",
    "params": { "id": "task_12345" }
  }'
```

### Cancel Task

Cancel an in-progress task:

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 7,
    "method": "CancelTask",
    "params": { "id": "task_12345" }
  }'
```

Response:
```json
{
  "jsonrpc": "2.0",
  "id": 7,
  "result": {
    "id": "task_12345",
    "contextId": "ctx_...",
    "status": {
      "state": "TASK_STATE_CANCELED",
      "timestamp": "2026-06-26T16:00:00.000Z"
    }
  }
}
```

## Links

- **API docs:** https://mdapi.io
- **MCP server manifest:** https://mdapi.io/mcp
- **Health check:** https://mdapi.io/health
- **API documentation:** https://mdapi.io/llms.txt
- **Full API documentation:** https://mdapi.io/llms-full.txt
- **AI discovery:** https://mdapi.io/.well-known/ai-discovery.json                                        or https://mdapi.io/ai-discovery.json
- **AI Agent discovery:** https://mdapi.io/.well-known/agent.json                                         or https://mdapi.io/agent.json
- **A2A Agent card:** https://mdapi.io/.well-known/agent-card.json                                        or https://mdapi.io/agent-card.json
- **ACP manifest:** https://mdapi.io/.well-known/acp.json                                                 or https://mdapi.io/acp.json
- **x402 payment manifest:** https://mdapi.io/.well-known/x402.json                                       or https://mdapi.io/x402.json
- **OpenAPI specification (JSON):** https://mdapi.io/.well-known/openapi.json                             or https://mdapi.io/openapi.json
- **OpenAPI specification (YAML):** https://mdapi.io/.well-known/openapi.yaml                             or https://mdapi.io/openapi.yaml
- **MAPI specification (case-insensitive path MAPI.md support):** https://mdapi.io/.well-known/mapi.md    or https://mdapi.io/mapi.md
- **Skill specification (case-insensitive path SKILL.md support):** https://mdapi.io/.well-known/skill.md or https://mdapi.io/skill.md

## External Links

- **github.com** https://github.com/mdapiio/mdapi.io
- **skills.sh** https://www.skills.sh/mdapiio/mdapi.io
- **clawhub.ai** https://clawhub.ai/mdapiio

## Disclaimer

**The service is provided "AS IS".**

