[![skills.sh](https://skills.sh/b/mdapiio/mdapi.io)](https://skills.sh/mdapiio/mdapi.io)

# mdapi.io - Minimal Data API I/O: a content transformation layer primitive for AI systems.


Transforms documents, images, and webpages into AI-ready Markdown and structured data, optimized for LLM efficiency and token usage.

## Agent entrypoint

- **Start AI discovery** → https://mdapi.io/.well-known/ai-discovery.json
- **Use skill** → https://mdapi.io/.well-known/skill.md

## Quick Start

Choose your entry point based on your role:

| Role                                                  | Protocol                     | Endpoint                  | When to use                                                                                                  |
| ----------------------------------------------------- | ---------------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------ |
| IDE / coding agent (JetBrains, Cursor, VS Code, etc.) | ACP (Agent Client Protocol)  | POST /acp                 | You are an IDE plugin or coding agent. Use tools/call with convert tool.                                     |
| AI agent (Claude Code, Codex, OpenClaw, Hermes, etc.) | A2A (Agent-to-Agent)         | POST /a2a                 | You are an autonomous agent. Use message/send with data in text parts. Supports streaming and task tracking. |
| AI agent (any framework)                              | MCP (Model Context Protocol) | GET  /mcp + POST /mcp     | You need tool discovery. Use tools/call with convert tool.                                                   |
| OpenAI-compatible client                              | OpenAI API                   | POST /v1/chat/completions | You already use OpenAI SDK. Pass URL/file in messages. Supports streaming.                                   |
| Direct HTTP / curl / script                           | REST API                     | GET  / or POST /          | Simplest path. GET returns Markdown directly. POST returns JSON with metadata.                               |

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
| --------- | ------------------------------ |
| Documents | PDF, DOCX, XLSX, XLS, ODT, ODS |
| Images    | JPEG, JPG, PNG, WebP, SVG      |
| Text      | HTML, XML, JSON, CSV, TXT      |
| Webpages  | Any publicly accessible URL    |

## Source Parameters (all protocols)

Every protocol (REST, MCP, ACP, A2A, OpenAI) converges on the **same conversion core**, so content is specified via a single unified `input` parameter everywhere.

| Parameter | Type   | Description                                                                                                                 |
| --------- | ------ | --------------------------------------------------------------------------------------------------------------------------- |
| `input`   | string | Content to convert. Auto-detected: starts with `http(s)://` → URL, starts with `data:` → file (data URI), otherwise → text. |

The `input` parameter is auto-detected by the core: URLs (starting with `http://` or `https://`) are fetched, data URIs (starting with `data:`) are decoded as files, and anything else is treated as raw text.
Additional parameters (`prompt`, `result`, `stream`, `token`, `memo`) are orthogonal and may be combined with `input`.

All five protocols expose the same `input` source and apply the same transformations, streaming, and prompt-driven processing - the only
difference is the transport (REST query/JSON, MCP `tools/call`, ACP RPC, OpenAI `messages`, A2A `message.parts`).

## Limits

| Limit               | Value                                                                            |
| ------------------- | -------------------------------------------------------------------------------- |
| **Max file size**   | 50 MB                                                                            |
| **Max URL content** | 50 MB                                                                            |
| **Rate limit**      | 10,000 requests per hour                                                         |
| **Free tier**       | 10 requests per day (no token required), within the service’s overall free quota |
| **Paid tier**       | min $0.01 per conversion (USDC on Solana)                                        |
| **Token validity**  | 1 year                                                                           |

## Authentication

**Recommended:** Use `Authorization: Bearer TOKEN`

| Method               | Use Case                                                 |
| -------------------- | -------------------------------------------------------- |
| Bearer (recommended) | `-H "Authorization: Bearer TOKEN"`                       |
| Header (x402)        | `-H "X-Token-Required: TOKEN"` (for x402 legacy clients) |
| Query (legacy)       | `?token=TOKEN` (for simple GET requests)                 |

## Token Activation

Before you can use a paid token, you must receive a 402 response first:

| Step | Description                                             |
| ---- | ------------------------------------------------------- |
| 1    | Request without token → Receive 402 with NEW token+memo |
| 2    | Send USDC on Solana to wallet with memo from 402        |
| 3    | Retry with EXACT token+memo from 402 → Activation       |
| 4    | After: use token only (no memo needed)                  |

Important: The token+memo issued in the 402 response MUST be used exactly. Using old token or different memo will be rejected.

## API Usage

### GET / (Content conversion)

Simple content conversion using query parameters. Returns Markdown directly.

#### Parameters

| Parameter | Type    | Required | Description                                                 |
| --------- | ------- | -------- | ----------------------------------------------------------- |
| `input`   | string  | *        | Content to convert (URL, text, or data URI — auto-detected) |
| `prompt`  | string  |          | Custom instructions for LLM processing                      |
| `result`  | string  |          | Response format: `markdown`, `prompt`, or `both`            |
| `stream`  | boolean |          | Enable streaming: true for SSE response                     |
| `token`   | string  |          | Access token for paid tier                                  |
| `memo`    | string  |          | Memo for token activation                                   |

*The `input` parameter is required.*

### POST / (Content conversion via JSON)

Supports content conversion via JSON body. The `input` parameter accepts URLs, text, or data URIs (auto-detected). Returns a JSON object containing the Markdown content.

#### Parameters

| Parameter | Type    | Required | Description                                                 |
| --------- | ------- | -------- | ----------------------------------------------------------- |
| `input`   | string  | *        | Content to convert (URL, text, or data URI — auto-detected) |
| `prompt`  | string  |          | Custom instructions for LLM processing                      |
| `result`  | string  |          | Response format: `markdown`, `prompt`, or `both`            |
| `stream`  | boolean |          | Enable streaming: true for SSE response                     |
| `token`   | string  |          | Access token for paid tier                                  |
| `memo`    | string  |          | Memo for token activation                                   |

*The `input` parameter is required.*

### Result Format Parameter

The `result` parameter controls the response format for both GET and POST requests.

| Value                | Description                                                     |
| -------------------- | --------------------------------------------------------------- |
| `markdown` (default) | Returns the converted Markdown content                          |
| `prompt`             | Returns the result of LLM processing with `prompt` instructions |
| `both`               | Returns both `markdown` and `prompt_result` in the response     |

When `result=both`:

- **GET requests** return Markdown combining `markdown`, followed by "## Prompt Result" and `prompt_result` (always in Markdown format)
- **POST requests** return JSON with `markdown` and `prompt_result` fields

> **Auto `result=prompt`:** When `prompt` is provided without an explicit `result`, the core automatically sets `result="prompt"` — so the response contains only the LLM-processed output. To get both Markdown and prompt result, set `result="both"`.

### Prompt Parameter

The `prompt` parameter lets you specify custom instructions for the LLM to follow when generating the result.

| Use Case           | Example                                                                           |
| ------------------ | --------------------------------------------------------------------------------- |
| Summarize document | `?input=https://example.com/doc.pdf&prompt=Summarize this document&result=prompt` |
| Extract key points | `?input=Your+text+here&prompt=Extract+key+points&result=prompt`                   |
| Convert to JSON    | `?input=https://example.com/doc.pdf&prompt=Convert+to+JSON+format&result=prompt`  |
| Analyze content    | `?input=Your+text+here&prompt=Analyze+and+explain&result=prompt`                  |

### Streaming Parameter

The `stream` parameter enables Server-Sent Events (SSE) streaming for real-time response delivery.

**Type:** `boolean`
**Default:** `false` (non-streaming)

Example:
```bash
curl "https://mdapi.io/?input=...&stream=true"
```

Response format (OpenAI-compatible SSE, one JSON object per `data:` line):
```json
data: {"type":"token_info","status":"valid","balance":99,"expires":1798761600,"resource":"https://example.com/file.pdf","mimetype":"application/pdf"}
data: {"choices":[{"index":0,"delta":{"content":" chunk"},"finish_reason":null}]}
data: {"choices":[{"index":0,"delta":{"content":" more"},"finish_reason":null}]}
data: {"choices":[{"index":0,"delta":{},"finish_reason":"stop"}]}
data: [DONE]
```

**Native streaming per protocol.** Every protocol delivers a *real* content stream when `stream: true`, but each emits it in its own native frame format (so existing clients keep working):

| Protocol | Streaming frame format                                                                                                            |
| -------- | --------------------------------------------------------------------------------------------------------------------------------- |
| REST     | OpenAI-compatible `choices/delta` frames                                                                                          |
| OpenAI   | `chat.completion.chunk` (`choices/delta`)                                                                                         |
| MCP      | `notifications/message` content chunks, then one final `tools/call` result frame                                                  |
| ACP      | incremental JSON-RPC `result.content` text chunks, then a final full `result` frame                                               |
| A2A      | incremental `task.artifacts[].parts[].text` chunks (`TASK_STATE_WORKING`), then a final completed `task` (`TASK_STATE_COMPLETED`) |

> **Note on MCP transport vs. the `stream` parameter.** The MCP manifest advertises `transport.type: "streamable-http"` - that is the MCP *transport*
> (how JSON-RPC requests are delivered to `POST /mcp`). It is unrelated to the `stream` *parameter*, which independently enables SSE streaming of the
> conversion **content**. You can use MCP without streaming; and when you do pass `stream: true`, the content arrives as SSE frames alongside the transport.

### Response Codes

| Code | Description       | Response Body (GET)                      | Response Body (POST)                                                                    |
| ---- | ----------------- | ---------------------------------------- | --------------------------------------------------------------------------------------- |
| 200  | Success           | Markdown content                         | JSON with `success`, markdown, prompt_result, resource, mimetype, metrics, token fields |
| 402  | Payment Required  | Markdown payment instructions            | JSON with `success:false` and payment object                                            |
| 400  | Bad Request       | Markdown error (`# Error\n\n...`)        | JSON `{"success":false,"error":"..."}`                                                  |
| 401  | Invalid Token     | Markdown error (`# Error\n\n...`)        | JSON `{"success":false,"error":"..."}`                                                  |
| 404  | Not Found         | Markdown error (`# Error\n\n...`)        | JSON `{"success":false,"error":"..."}`                                                  |
| 413  | Payload Too Large | Markdown error (`# Error\n\n...`)        | JSON `{"success":false,"error":"..."}`                                                  |
| 429  | Rate Limited      | Markdown error (`# Error\n\n...`)        | JSON `{"success":false,"error":"..."}`                                                  |
| 500  | Server Error      | Markdown error (`# Server Error\n\n...`) | JSON `{"success":false,"error":"..."}`                                                  |

### Token Status

The X-Token-Status header (and token_status field in responses) indicates the current state of authentication:

| Status             | Description                                                                             |
| ------------------ | --------------------------------------------------------------------------------------- |
| free               | Free tier (no token required, 10 requests/day), within the service's overall free quota |
| valid              | Paid token active with remaining balance                                                |
| invalid            | Token not found or not provided                                                         |
| expired            | Token validity period has ended                                                         |
| exhausted          | Token balance has been fully used                                                       |
| expired_pending    | Activation memo has expired                                                             |
| activated          | Token was just activated with this request                                              |
| verification_error | Payment verification failed                                                             |
| invalid_payment    | Payment transaction is invalid                                                          |
| error              | Internal error during token processing                                                  |
| pending            | Payment required (token not yet activated)                                              |

### Endpoints

| Method | Path                                                 | Description                                                                 |
| ------ | ---------------------------------------------------- | --------------------------------------------------------------------------- |
| GET    | /about                                               | About service                                                               |
| GET    | /                                                    | API docs or conversion (via query parameter: `input`, `prompt`, `result`)   |
| POST   | /                                                    | Convert content via JSON body (supports `input`, `prompt`, `result` params) |
| POST   | /v1/chat/completions                                 | OpenAI-compatible endpoint                                                  |
| GET    | /mcp                                                 | MCP server manifest                                                         |
| POST   | /mcp                                                 | MCP RPC endpoint (discover, tools, resources, subscriptions)                |
| POST   | /acp                                                 | ACP RPC endpoint (IDE agents)                                               |
| POST   | /a2a                                                 | A2A RPC endpoint (agent2agent)                                              |
| GET    | /health                                              | Health check                                                                |
| GET    | /llms.txt                                            | API documentation                                                           |
| GET    | /llms-full.txt                                       | Full API documentation                                                      |
| GET    | /.well-known/ai-discovery.json or /ai-discovery.json | AI discovery                                                                |
| GET    | /.well-known/agent.json        or /agent.json        | AI Agent discovery                                                          |
| GET    | /.well-known/agent-card.json   or /agent-card.json   | A2A Agent card                                                              |
| GET    | /.well-known/acp.json          or /acp.json          | ACP manifest                                                                |
| GET    | /.well-known/x402.json         or /x402.json         | x402 payment manifest                                                       |
| GET    | /.well-known/openapi.json      or /openapi.json      | OpenAPI specification (JSON)                                                |
| GET    | /.well-known/openapi.yaml      or /openapi.yaml      | OpenAPI specification (YAML)                                                |
| GET    | /.well-known/mapi.md           or /mapi.md           | MAPI specification (case-insensitive path MAPI.md support)                  |
| GET    | /.well-known/skill.md          or /skill.md          | Skill specification (case-insensitive path SKILL.md support)                |

#### Examples

```bash
# URL conversion via GET (free)
curl "https://mdapi.io/?input=https://example.com/file.pdf"

# URL with prompt and result=both (returns markdown + prompt_result)
curl "https://mdapi.io/?input=https://example.com/file.pdf&prompt=Summarize&result=both"

# Text with prompt and result=prompt (return prompt_result)
curl "https://mdapi.io/?input=Your+text+here&prompt=Summarize+this&result=prompt"

# Token activation via GET (activate and use)
curl "https://mdapi.io/?input=https://example.com/file.pdf&token=YOUR_TOKEN&memo=YOUR_MEMO"

# Paid request with token via GET (using token)
curl "https://mdapi.io/?input=https://example.com/file.pdf&token=YOUR_TOKEN"

# URL conversion via POST (free)
curl -X POST -H "Content-Type: application/json" -d '{"input":"https://example.com/file.pdf"}' "https://mdapi.io/"

# Text with prompt via POST
curl -X POST -H "Content-Type: application/json" -d '{"input":"Your text here","prompt":"Summarize","result":"both"}' "https://mdapi.io/"

# File upload via POST (data URI)
curl -X POST -H "Content-Type: application/json" -d '{"input":"data:application/pdf;base64,JVBERi0xLjQK..."}' "https://mdapi.io/"

# Token activation via POST
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer YOUR_TOKEN" -H "X-Memo-Required: YOUR_MEMO" -d '{"input":"https://example.com/file.pdf"}' "https://mdapi.io/"

# Paid request with token via POST
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer YOUR_TOKEN" -d '{"input":"https://example.com/file.pdf"}' "https://mdapi.io/"

```

### OpenAI Compatible Endpoint

The `/v1/chat/completions` endpoint provides an OpenAI‑compatible API for markdown conversion with streaming support.

**Supported features:**
- URL extraction from message content (any text containing https?://)
- image_url in messages (OpenAI format) - supports HTTP URLs and data URLs
- file in messages (OpenAI format) - base64 encoded files (field `file.data`, `file.filename`, `file.mimeType`)
- Direct text content in messages (any text without a URL is sent to the core as the `input` source and converted to Markdown)
- Token and memo via headers (recommended for POST)
- Streaming SSE responses (`stream: true`)
- Custom instructions with LLM processing (system messages, or user messages containing instruction keywords such as *extract, summarize, analyze, format, convert to, write as, create, generate, json* → LLM-driven summary/extraction/transformation)
- `prompt` + `result` for LLM-processed output. The response surfaces `prompt_result`, `resource`, and `mimetype` at the top level alongside the standard `choices[].message.content` (which carries `prompt_result` for `result=prompt`, otherwise the Markdown).

`model` is accepted but not required (any string; the service uses its own conversion pipeline, not a remote LLM chat model, unless custom instructions trigger LLM processing).

> **Content via message text:** the message text is passed to the core as the `input` parameter - the same unified source as every other protocol. URLs are auto-detected, data URIs are decoded as files, and plain text is processed directly.
> See [Source Parameters (all protocols)](#source-parameters-all-protocols).

#### Request Schema

```json
{
  "type": "object",
  "properties": {
    "model": {
      "type": "string",
      "description": "Optional model identifier (any string accepted; not required)"
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
    },
    "prompt": {
      "type": "string",
      "description": "Custom instructions for LLM processing (alternative to instruction keywords in messages)"
    },
    "result": {
      "type": "string",
      "enum": ["markdown", "prompt", "both"],
      "description": "Response format when using prompt"
    },
    "token": {"type": "string", "description": "Access token for paid tier"},
    "memo": {"type": "string", "description": "Memo for token activation"},
    "input": {
      "type": "string",
      "description": "Content to convert (URL, text, or data URI — auto-detected). Alternative to a URL/file embedded in messages"
    }
  },
  "required": ["messages"]
}
```

> **Note on streaming + custom instructions:** when custom instructions trigger LLM processing, the response is returned as a single completion (streaming is not applied to the LLM pass). Streaming SSE applies to the standard conversion path.

## MCP Configuration

Connect mdapi.io to your MCP-compatible client (spec 2026-07-28, stateless).

> **Single source via `input`:** the `convert` tool accepts a unified `input` parameter — the same source as every other protocol. The core auto-detects whether the value is a URL, data URI, or text.
> See [Source Parameters (all protocols)](#source-parameters-all-protocols).

### Protocol Requirements

- **Transport:** Streamable HTTP (POST-only for JSON-RPC, GET for manifest)
- **Required headers:** `MCP-Protocol-Version: 2026-07-28`, `Mcp-Method`, `Mcp-Name`
- **Stateless:** No sessions - every request is independent
- **Discovery:** Use `server/discover` to query server capabilities and supported versions

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

> **Note:** No token is required to connect. A free tier is available (10 requests per day), within the service’s overall free quota.

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

MCP does not use HTTP-level Authorization headers. The token is always passed inside the tool `arguments` object.

**Activation** — include `token` + `memo` in the first request:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "input": "https://example.com/doc.pdf",
      "token": "YOUR_TOKEN",
      "memo": "YOUR_PAYMENT_MEMO"
    }
  }
}
```

**After activation** — use `token` only (no memo needed):

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "input": "https://example.com/doc.pdf",
      "token": "YOUR_ACTIVATED_TOKEN"
    }
  }
}
```

### MCP Tool Examples

Convert with prompt and result:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "input": "https://example.com/doc.pdf",
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
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "input": "Your text content here",
      "prompt": "Extract key points",
      "result": "prompt"
    }
  }
}
```

Stream with SSE (native MCP frames):

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "input": "https://example.com/doc.pdf",
      "stream": true
    }
  }
}
```

Response (SSE over `streamable-http`): intermediate `notifications/message` content chunks, then one final `tools/call` result frame with the full Markdown:

```
data: {"jsonrpc":"2.0","method":"notifications/message","params":{"level":"info","data":" partial "}}
data: {"jsonrpc":"2.0","method":"notifications/message","params":{"level":"info","data":" more "}}
data: {"jsonrpc":"2.0","id":3,"result":{"content":[{"type":"text","text":"<full converted content>"}],"isError":false}}
data: [DONE]
```

### Using Environment Variables

Or use environment variable:

```bash
export MDAPI_TOKEN=YOUR_ACTIVATED_TOKEN
```

## Code Examples

### JavaScript (fetch)

```javascript
// Convert a URL via GET — returns Markdown directly
const response = await fetch('https://mdapi.io/?input=https://example.com/doc.pdf');
const markdown = await response.text();
console.log(markdown);
```

```javascript
// Convert a URL via POST — returns JSON with metadata
const response = await fetch('https://mdapi.io/', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ input: 'https://example.com/doc.pdf' })
});
const data = await response.json();
console.log(data.markdown);
```

```javascript
// Text with prompt — returns prompt_result
const response = await fetch('https://mdapi.io/', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    input: 'Your text here',
    prompt: 'Summarize this',
    result: 'both'
  })
});
const data = await response.json();
console.log(data.markdown);
console.log(data.prompt_result);
```

```javascript
// File upload via data URI
const response = await fetch('https://mdapi.io/', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    input: 'data:application/pdf;base64,JVBERi0xLjQK...'
  })
});
const data = await response.json();
console.log(data.markdown);
```

```javascript
// Token activation — first request with token + memo
const response = await fetch('https://mdapi.io/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer YOUR_TOKEN',
    'X-Memo-Required': 'YOUR_MEMO'
  },
  body: JSON.stringify({ input: 'https://example.com/doc.pdf' })
});
const data = await response.json();
// After activation, use token only (no memo needed)
```

```javascript
// Streaming via OpenAI-compatible endpoint
const response = await fetch('https://mdapi.io/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer YOUR_TOKEN'
  },
  body: JSON.stringify({
    model: 'markdown-v1',
    messages: [{ role: 'user', content: 'Convert https://example.com/doc.pdf' }],
    stream: true
  })
});

const reader = response.body.getReader();
const decoder = new TextDecoder();
while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  process.stdout.write(decoder.decode(value));
}
```

### Python

```python
import requests

# Convert a URL via GET — returns Markdown directly
response = requests.get('https://mdapi.io/?input=https://example.com/doc.pdf')
response.raise_for_status()
print(response.text)
```

```python
import requests

# Convert a URL via POST — returns JSON with metadata
response = requests.post(
    'https://mdapi.io/',
    json={'input': 'https://example.com/doc.pdf'}
)
response.raise_for_status()
data = response.json()
print(data['markdown'])
```

```python
import requests

# Text with prompt — returns prompt_result
response = requests.post(
    'https://mdapi.io/',
    json={'input': 'Your text', 'prompt': 'Summarize', 'result': 'both'}
)
response.raise_for_status()
data = response.json()
print(data['markdown'])
print(data['prompt_result'])
```

```python
import requests

# File upload via data URI
with open('document.pdf', 'rb') as f:
    import base64
    file_data = base64.b64encode(f.read()).decode()
    response = requests.post(
        'https://mdapi.io/',
        json={'input': f'data:application/pdf;base64,{file_data}'}
    )
    response.raise_for_status()
    data = response.json()
    print(data['markdown'])
```

```python
import requests

# Token activation
response = requests.post(
    'https://mdapi.io/',
    json={'input': 'https://example.com/doc.pdf'},
    headers={
        'Authorization': 'Bearer YOUR_TOKEN',
        'X-Memo-Required': 'YOUR_MEMO'
    }
)
response.raise_for_status()
# After activation, use token only (no memo needed)
```

```python
from openai import OpenAI

# OpenAI-compatible streaming
client = OpenAI(base_url='https://mdapi.io/v1', api_key='YOUR_TOKEN')
stream = client.chat.completions.create(
    model='markdown-v1',
    messages=[{'role': 'user', 'content': 'Convert https://example.com/doc.pdf'}],
    stream=True
)
for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end='')
```

### Go

```go
package main

import (
    "fmt"
    "io"
    "net/http"
)

func main() {
    // Convert a URL via GET — returns Markdown directly
    resp, err := http.Get("https://mdapi.io/?input=https://example.com/doc.pdf")
    if err != nil {
        fmt.Println("HTTP error:", err)
        return
    }
    defer resp.Body.Close()

    body, _ := io.ReadAll(resp.Body)
    fmt.Println(string(body))
}
```

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
)

func main() {
    // Convert a URL via POST — returns JSON with metadata
    payload, _ := json.Marshal(map[string]string{
        "input": "https://example.com/doc.pdf",
    })

    resp, err := http.Post(
        "https://mdapi.io/",
        "application/json",
        bytes.NewReader(payload),
    )
    if err != nil {
        fmt.Println("HTTP error:", err)
        return
    }
    defer resp.Body.Close()

    var result map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&result)
    fmt.Println(result["markdown"])
}
```

```go
package main

import (
    "bytes"
    "encoding/base64"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
)

func main() {
    // File upload via data URI
    fileBytes, _ := os.ReadFile("document.pdf")
    b64 := base64.StdEncoding.EncodeToString(fileBytes)

    payload, _ := json.Marshal(map[string]string{
        "input": "data:application/pdf;base64," + b64,
    })

    resp, err := http.Post(
        "https://mdapi.io/",
        "application/json",
        bytes.NewReader(payload),
    )
    if err != nil {
        fmt.Println("HTTP error:", err)
        return
    }
    defer resp.Body.Close()

    var result map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&result)
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
    let url = "https://mdapi.io/?input=https://example.com/doc.pdf";

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
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Deserialize)]
struct ApiResponse {
    markdown: Option<String>,
    prompt_result: Option<String>,
}

#[derive(Serialize)]
struct ConvertRequest {
    input: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    prompt: Option<String>,
    #[serde(skip_serializing_if = "Option::is_none")]
    result: Option<String>,
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
        let body = ConvertRequest {
            input: url.to_string(),
            prompt: prompt.map(|p| p.to_string()),
            result: prompt.map(|_| "both".to_string()),
        };

        let mut request = self.client
            .post(&self.base_url)
            .header("Content-Type", "application/json")
            .json(&body);

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

// Example: convert a URL with prompt
#[tokio::main]
async fn main() -> Result<()> {
    let token = std::env::var("MDAPI_TOKEN").ok();
    let client = MdApiClient::new(token);
    let markdown = client
        .convert_url("https://example.com/doc.pdf", Some("Summarize"))
        .await?;
    println!("{}", markdown);
    Ok(())
}
```

```rust
use anyhow::Result;
use reqwest::Client;
use base64::engine::general_purpose::STANDARD;
use base64::Engine;

#[tokio::main]
async fn main() -> Result<()> {
    let client = Client::new();
    let file_bytes = std::fs::read("document.pdf")?;
    let b64 = STANDARD.encode(&file_bytes);

    let body = serde_json::json!({
        "input": format!("data:application/pdf;base64,{}", b64)
    });

    let response = client
        .post("https://mdapi.io/")
        .header("Content-Type", "application/json")
        .json(&body)
        .send()
        .await?;

    let data: serde_json::Value = response.json().await?;
    println!("{}", data["markdown"]);
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

#### OpenAI with paid token

```bash
curl -X POST "https://mdapi.io/v1/chat/completions"   -H "Authorization: Bearer YOUR_TOKEN"   -H "X-Memo-Required: YOUR_MEMO"   -H "Content-Type: application/json"   -d '{"model":"markdown-v1","messages":[{"role":"user","content":"Convert https://example.com/doc.pdf"}]}'
```

After activation, use token only (no memo needed):

```bash
curl -X POST "https://mdapi.io/v1/chat/completions"   -H "Authorization: Bearer YOUR_ACTIVATED_TOKEN"   -H "Content-Type: application/json"   -d '{"model":"markdown-v1","messages":[{"role":"user","content":"Convert https://example.com/doc.pdf"}]}'
```

## A2A Configuration

Connect mdapi.io to your A2A-compatible agent (Claude Code, Codex, OpenClaw, Hermes, etc.).

### Basic Configuration

Add to your A2A client configuration:

```json
{
  "agent": {
    "name": "mdapi",
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
  "method": "message/send",
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

| Method            | Description                             |
| ----------------- | --------------------------------------- |
| message/send      | Send a message to initiate conversion   |
| message/stream    | Send message with SSE streaming updates |
| tasks/get         | Get task status and results by ID       |
| tasks/list        | List tasks with optional filtering      |
| tasks/cancel      | Cancel an in-progress task              |
| tasks/resubscribe | Subscribe to task updates via SSE       |

> **Single source via `input`:** the `input` parameter in the message parts is the unified source — the same as the REST endpoint. A bare URL
> inside a text part (e.g. `"Convert https://example.com/doc.pdf"`) is extracted automatically and used as the conversion source, so you don't need to wrap it
> in structured JSON. Instructions such as `Summarize` should be passed via the structured `{ "input": "...", "prompt": "..." }` form, not mixed into the text.

### A2A Examples

#### message/send

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "message/send",
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

#### message/send with file (data URI)

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "message/send",
    "params": {
      "message": {
        "messageId": "msg-uuid-2",
        "parts": [
          { "text": "{"input":"data:application/pdf;base64,JVBERi0xLjQK..."}" }
        ]
      }
    }
  }'
```

#### message/send with structured data

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "message/send",
    "params": {
      "message": {
        "messageId": "msg-uuid-3",
        "parts": [
          {
            "data": {
              "input": "https://example.com/doc.pdf",
              "result": "markdown"
            },
            "mediaType": "application/json"
          }
        ]
      }
    }
  }'
```

#### Token activation via A2A

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 7,
    "method": "message/send",
    "params": {
      "message": {
        "messageId": "msg-uuid-7",
        "parts": [
          {
            "data": {
              "input": "https://example.com/doc.pdf",
              "token": "YOUR_TOKEN",
              "memo": "YOUR_PAYMENT_MEMO"
            },
            "mediaType": "application/json"
          }
        ]
      }
    }
  }'
```

> **Note on token activation:** Pass `token` and `memo` inside a `data` Part or as JSON inside a `text` Part. A2A does not use HTTP-level Authorization headers.

#### Multi-turn conversation (follow-up)

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 4,
    "method": "message/send",
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

#### tasks/get

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 5,
    "method": "tasks/get",
    "params": { "id": "task_12345" }
  }'
```

#### tasks/list

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 6,
    "method": "tasks/list",
    "params": { "contextId": "ctx_12345", "pageSize": 10 }
  }'
```

### A2A Message Parts

Messages use the A2A `Part` format (field-name discriminators per spec v1.0.0):

| Type   | Description                               | Fields                                  |
| ------ | ----------------------------------------- | --------------------------------------- |
| `text` | Plain text content or JSON-encoded params | `text`                                  |
| `data` | Structured JSON data (core params)        | `data` (object), `mediaType` (optional) |
| `url`  | URL to fetch and convert                  | `url` (http/https)                      |

**Part → Core Parameter Mapping:**
- `text` Part → `input` param (direct content) or JSON-encoded params (`{ "input": "...", "prompt": "..." }`)
- `data` Part → merged as params (`input`, `prompt`, `result`, `token`, `memo`, etc.)
- `url` Part → `input` param (fetched and converted)

### Message Object

```typescript
interface Message {
  messageId: string;                  // REQUIRED: unique ID (e.g. "msg-uuid")
  contextId?: string;                 // Optional: group related tasks
  taskId?: string;                    // Optional: associate with existing task
  role: "user" | "agent";   // REQUIRED
  parts: Array<Part>;                 // REQUIRED: at least one part
}
```

### Task Data Model

```typescript
interface Task {
  id: string;                         // "task_<timestamp>_<random>"
  contextId: string;                  // "ctx_<timestamp>_<random>"
  status: {
    state: string;                    // "working" | "completed" | "failed" | "canceled" | "rejected"
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

| State       | Description                   |
| ----------- | ----------------------------- |
| `working`   | Task is being processed       |
| `completed` | Task finished successfully    |
| `failed`    | Task failed during processing |
| `canceled`  | Task was canceled by client   |
| `rejected`  | Task was rejected by server   |

### Error Responses

A2A uses JSON-RPC 2.0 error format with A2A-specific error codes:

| Code     | Error               | Description                                                |
| -------- | ------------------- | ---------------------------------------------------------- |
| `-32700` | Parse error         | Invalid JSON payload                                       |
| `-32600` | Invalid Request     | Missing required fields (message.parts, message.messageId) |
| `-32601` | Method not found    | Unknown A2A method                                         |
| `-32001` | Task not found      | Task ID does not exist                                     |
| `-32002` | Task not cancelable | Task in terminal state                                     |
| `-32003` | Cannot subscribe    | Task in terminal state                                     |

Unsupported media type is returned as HTTP **415** (not a JSON-RPC error code).

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

Use `message/stream` for real-time SSE updates:

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -H "Accept: text/event-stream"   -d '{
    "jsonrpc": "2.0",
    "id": 5,
    "method": "message/stream",
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

Response format (real content stream; intermediate `working` chunks, then the final completed `task`):

```
data: {"jsonrpc":"2.0","id":5,"result":{"task":{"id":"task_...","contextId":"ctx_...","status":{"state":"working"},"artifacts":[{"artifactId":"artifact_...","name":"conversion_result","parts":[{"text":" partial "}]}]}}}
data: {"jsonrpc":"2.0","id":5,"result":{"task":{"id":"task_...","contextId":"ctx_...","status":{"state":"working"},"artifacts":[{"artifactId":"artifact_...","name":"conversion_result","parts":[{"text":" more "}]}]}}}
data: {"jsonrpc":"2.0","id":5,"result":{"task":{"id":"task_...","contextId":"ctx_...","status":{"state":"completed"},"artifacts":[{"artifactId":"artifact_...","name":"conversion_result","parts":[{"text":"<full converted content>"}]}]}}}
data: [DONE]
```

### Subscribe to Task

Subscribe to an existing task for real-time updates:

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -H "Accept: text/event-stream"   -d '{
    "jsonrpc": "2.0",
    "id": 6,
    "method": "tasks/resubscribe",
    "params": { "id": "task_12345" }
  }'
```

### Cancel Task

Cancel an in-progress task:

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 7,
    "method": "tasks/cancel",
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
      "state": "canceled",
      "timestamp": "2026-06-26T16:00:00.000Z"
    }
  }
}
```

## ACP Configuration

Connect mdapi.io to your IDE or coding agent (JetBrains, Cursor, VS Code, etc.) via the Agent Client Protocol. ACP is a JSON-RPC 2.0 endpoint at `POST /acp`.

> **Single source via `input`:** the `convert` tool accepts a unified `input` parameter — the same source as every other protocol. The core auto-detects whether the value is a URL, data URI, or text.
> See [Source Parameters (all protocols)](#source-parameters-all-protocols).

### Basic Configuration

Add to your ACP client configuration (IDE plugin / agent settings):

```json
{
  "acpServers": {
    "mdapi": {
      "url": "https://mdapi.io/acp"
    }
  }
}
```

Or call the JSON-RPC endpoint directly with `POST /acp` (Content-Type: `application/json`):

```bash
curl -X POST https://mdapi.io/acp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "convert",
      "arguments": {
        "input": "https://example.com/doc.pdf"
      }
    }
  }'
```

> **Note:** `GET /acp` is not supported (ACP is POST-only). A free tier is available without a token.

### ACP Methods

| Method           | Description                               |
| ---------------- | ----------------------------------------- |
| `tools/list`     | List available tools (includes `convert`) |
| `tools/call`     | Call the `convert` tool                   |
| `resources/list` | List available discovery resources        |
| `resources/read` | Read a discovery resource by URI          |

### ACP Tool Examples

**URL conversion:**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "input": "https://example.com/doc.pdf",
      "prompt": "Summarize this document",
      "result": "both",
      "stream": true
    }
  }
}
```

**Text conversion (prompt result):**

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "input": "Your text content here",
      "prompt": "Extract key points",
      "result": "prompt",
      "stream": false
    }
  }
}
```

**File conversion (data URI — auto-detected):**

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "input": "data:application/pdf;base64,JVBERi0xLjQK...",
      "prompt": "Extract the title",
      "result": "markdown",
      "stream": true
    }
  }
}
```

**Token activation (first request only):**

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "input": "https://example.com/doc.pdf",
      "token": "YOUR_NEW_TOKEN",
      "memo": "YOUR_PAYMENT_MEMO"
    }
  }
}
```

### ACP Response Format

ACP returns a JSON-RPC 2.0 result with a `content` array. The `convert` tool maps the unified core response so Markdown, the prompt result, and the source resource all surface as content items:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      { "type": "text", "text": "<converted Markdown>" },
      { "type": "text", "text": "<prompt_result, when result=prompt|both>" },
      { "type": "resource_link", "uri": "https://example.com/doc.pdf", "name": "https://example.com/doc.pdf" }
    ],
    "isError": false
  }
}
```

- `content[]` holds `text` items for `markdown` and `prompt_result`, plus a `resource_link` when the source is a URL.
- `isError` is `true` only when the underlying conversion failed (non-200 or `success: false`).
- When `stream: true`, the response is delivered as SSE chunks instead of a single JSON object. Each chunk is a JSON-RPC `result` with an incremental `content` text item, followed by a final full `result` frame (see [Streaming Parameter](#streaming-parameter)):

```
data: {"jsonrpc":"2.0","id":3,"result":{"content":[{"type":"text","text":" partial "}],"isError":false}}
data: {"jsonrpc":"2.0","id":3,"result":{"content":[{"type":"text","text":" more "}],"isError":false}}
data: {"jsonrpc":"2.0","id":3,"result":{"content":[{"type":"text","text":"<full converted content>"}],"isError":false}}
data: [DONE]
```

- Token status (`X-Token-Balance`, `X-Token-Expires`, `X-Token-Status`) is proxied from the core into both the response body and headers.

## Usage scenarios

mdapi.io is a minimal, self-documenting service-transport primitive: REST, MCP, ACP, A2A, and OpenAI-compatible endpoints all call the same transformation core, so agents can combine protocols and pass already-processed knowledge between each other.

- **Agent swarms** - each request is handled by a stateless, automatically-scaled execution environment, so the service scales horizontally. An orchestrator fans work out across a swarm of agents, and the swarm processes very large batches of distinct resources in parallel - millions of resources in a matter of minutes, the ceiling set by how widely the work is distributed rather than by the service. Different users may freely access the same resource.
- **Shared vs individual payment** - an orchestrator can pay once for a shared token (batching on-chain activity), or each agent can activate its own token for the exact volume it received.
- **Human-in-the-loop** - if an agent has no wallet or insufficient funds, it returns payment details + a QR code; the human pays from a mobile device and the agent resumes.
- **Role switching** - an agent's role can change mid-task; one agent fetches/normalizes, another summarizes/extracts, relaying compact results via the text or prompt parameters.
- **Cross-protocol interoperability** - knowledge extracted on one protocol is reusable on another.
- **Bulk processing / model training** - the swarm pattern turns mdapi.io into a high-throughput edge pipeline for large corpora.

See https://mdapi.io/about for the full scenario walkthrough.

## Links

- **About service:** https://mdapi.io/about
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
- **x.com** https://x.com/mdapiio

## Disclaimer

**The service is provided "AS IS".**


> mdapi.io is an edge-native service-transport primitive for AI, autonomous-agents, and the Web4 ecosystem.
