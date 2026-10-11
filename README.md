[![skills.sh](https://skills.sh/b/mdapiio/mdapi.io)](https://skills.sh/mdapiio/mdapi.io)

# mdapi.io - Minimal Data API I/O: a content transformation layer primitive for AI systems.


Transforms documents, images, and webpages into AI-ready Markdown and structured data, optimized for LLM efficiency and token usage.

## Agent entrypoint

- **Start AI discovery** → https://mdapi.io/.well-known/ai-discovery.json
- **Use skill** → https://mdapi.io/.well-known/skill.md

## Quick Start

Choose your entry point based on your role:

| Role                                                  | Protocol                     | Endpoint                                       | When to use                                                                                                                              |
| ----------------------------------------------------- | ---------------------------- | ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| IDE / coding agent (JetBrains, Cursor, VS Code, etc.) | ACP (Agent Client Protocol)  | POST /acp                                      | You are an IDE plugin or coding agent. Use initialize + session/new + session/prompt (content arrives via session/update notifications). |
| AI agent (Claude Code, Codex, OpenClaw, Hermes, etc.) | A2A (Agent-to-Agent)         | POST /a2a                                      | You are an autonomous agent. Use SendMessage with data in text parts. Supports streaming and task tracking.                              |
| AI agent (any framework)                              | MCP (Model Context Protocol) | GET /mcp + POST /mcp                           | You need tool discovery. Use tools/call with convert tool.                                                                               |
| OpenAI-compatible client                              | OpenAI API                   | POST /v1/chat/completions + POST /v1/responses | You already use OpenAI SDK. Pass URL/file in messages. Supports streaming.                                                               |
| Direct HTTP / curl / script                           | REST API                     | GET / or POST /                                | Simplest path. GET returns Markdown directly. POST returns JSON with metadata.                                                           |

### Universal discovery

All protocols and capabilities are described in one file:
GET /.well-known/ai-discovery.json

## Features

- Stateless, in-memory processing
- Edge execution with automatic scaling
- Prompt-driven transformation
- AI-optimized output for LLMs
- Pay-per-use via x402 v1/v2 or manual payment

## Supported Formats

| Type          | Formats                                                                                                                                                                                                         |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Documents     | DOCX, DOC, DOT, WIZ, ODT, RTF, PDF                                                                                                                                                                              |
| Spreadsheets  | XLSX, XLS, XLT, ODS, XLSM, XLSB, ET, Numbers, DTA                                                                                                                                                               |
| Presentations | PPTX, PPT, POT, PPS, PWZ, ODP                                                                                                                                                                                   |
| Images        | JPEG, JPG, PNG, WebP, SVG, GIF, BMP                                                                                                                                                                             |
| Text          | HTML, XML, JSON, CSV, TSV, TXT, MD, YAML, TOML, JS, PY, PL, RB, GO, RS, C, CPP, CS, JAVA, PHP, CSS, SCSS, SASS, LESS, SQL, SH, PS1, BAT, ASM, TEX, RST, GRAPHQL, JSON5, HCL, LOG, CONF, INI, VTT, VCF, ICS, EML |
| E-books       | EPUB                                                                                                                                                                                                            |
| Archives      | ZIP, TAR, TGZ, 7Z, GZ, XZ, LZMA, BZ2, TBZ2, TBZ, ZST, TZST                                                                                                                                                      |
| Webpages      | Any publicly accessible URL                                                                                                                                                                                     |

### How a file is read

A file travels as a data URI (`data:<mime>;base64,...`) and the declared MIME is what the service reads - the file name is only consulted when the type is absent or carries no format, and failing that the leading bytes are inspected.

| Declared MIME in the data URI                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | How it is read                                                                                                                                                                                                         |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `application/pdf`, `application/vnd.openxmlformats-officedocument.wordprocessingml.document`, `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `application/vnd.ms-excel`, `application/vnd.ms-excel.sheet.macroenabled.12`, `application/vnd.ms-excel.sheet.binary.macroenabled.12`, `application/vnd.oasis.opendocument.text`, `application/vnd.oasis.opendocument.spreadsheet`, `application/vnd.apple.numbers`, `application/x-stata-dta`, `image/jpeg`, `image/png`, `image/webp`, `image/svg+xml`, `image/gif`, `image/bmp`, `text/html`, `application/xml`, `text/xml`, `text/csv`                    | Converted: a binary document, an image or marked-up text, answered as Markdown                                                                                                                                         |
| `text/plain`, `application/json`, `text/json`, `text/markdown`, `application/javascript`, `text/javascript`, `text/yaml`, `application/toml`, `text/x-python`, `text/x-perl`, `text/x-ruby`, `text/x-golang`, `text/x-rust`, `text/x-c`, `text/x-c++`, `text/x-csharp`, `text/x-java`, `text/x-php`, `text/css`, `text/x-scss`, `text/x-sass`, `text/x-less`, `text/x-sql`, `text/x-sh`, `text/x-powershell`, `text/x-msdos-batch`, `text/x-asm`, `text/x-tex`, `text/x-rst`, `application/graphql`, `application/json5`, `text/x-hcl`, `text/tab-separated-values`, `text/vtt`, `text/vcard`, `text/calendar`, `message/rfc822` | Read as text with no conversion: source code, configuration and markup, tabular text and structured data. The fenced code language comes from the MIME itself, so a `text/x-python` payload arrives fenced as `python` |

The MIME declared in the data URI decides how the bytes are read, not the file name: a type the service knows is used as given. Only when it is absent, or is a placeholder that carries no format, does the extension of the file name get a say, and if neither settles it the leading bytes are inspected. A document is accepted by its MIME wherever the format is supported, so `.pdf`, `.PDF` and a file with no name at all read the same when the type is declared. Formats the service reads itself - EPUB, legacy DOC/XLS/PPT, PPTX, RTF, the archives and the compression family - are resolved the same way.

### Page boilerplate is kept

A converted page keeps its navigation, banners, footer and cookie notices. This is deliberate: the conversion is lossless, so what the page says is what the answer says. Stripping boilerplate is a heuristic, and its failure mode is deleting real content while the response still reports success - a loss the client cannot see is a loss it cannot undo, while visible noise can be ignored.

## Source Parameters (all protocols)

Every protocol (REST, MCP, ACP, A2A, OpenAI) converges on the **same conversion core**, so content is specified via a single unified `input` parameter everywhere.

| Parameter | Type   | Description                                                                                                                                                                                                                                                 |
| --------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `input`   | string | Content to convert: a URL (http(s)://...), a data URI (data:mime/type;base64,...), plain text, or a resource address (mdapi:...) returned by an earlier answer. Auto-detected: http(s):// → URL, data: → file, mdapi: → resource address, otherwise → text. |

The `input` parameter is auto-detected by the core: URLs (starting with `http://` or `https://`) are fetched, data URIs (starting with `data:`) are decoded as files, a `resource_id` from an earlier answer (starting with `mdapi:`) is read as the document it addresses, and anything else is treated as raw text.
All other parameters are orthogonal and may be combined with `input`.

| Parameter  | Values                                                                           | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ---------- | -------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `result`   | `markdown`, `prompt`, `both`, `meta`                                             | What the answer is: `markdown` (the converted text layer), `prompt` (the model's answer to `prompt`), `both`, or `meta` - a reconnaissance answer that reports the parts of the document and their sizes, and no content: the shape of the answer, reported before the content itself is fetched.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `part`     | `1`..`parts.count`, `N-M`, or a comma-separated list                             | Serves the parts of a document instead of the whole answer, so a one-page request of a 500-page document sends one page to the model and returns one page. The unit is the document's natural boundary - page (PDF), slide (presentations), sheet (spreadsheets), chapter (EPUB and archives, an expanded `.tar.zst` included) - or one A4 sheet of text for formats without one; the units table says which format owns which. Every response self-describes the range in `parts` (JSON body) and `X-MDAPI-Parts` / `X-MDAPI-Part-Unit` (headers), so the count never has to be guessed. `part` also decides how a resource larger than the input gate is read - see the addressability table.                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `find`     | literal substrings, whitespace-separated (no wildcards, no regular expressions)  | Searches the converted document and answers with the ADDRESSES of the matches - the part number and the character offset inside that part - never the text itself, so a search costs one request instead of reading the document. Case is ignored, the match is literal, and matches do not overlap. Whitespace separates terms (`find=pay court`), and every term must occur in the same part, in any order, so the answer says where they meet; one term answers with every occurrence, as it always has. Composes with `part`: without it the whole document is searched, with it only the parts named (and each hit still carries its own part number). A needle that does not occur is an honest empty answer (`matches: 0`) - with several terms `terms[]` says which one is nowhere - and `truncated: true` marks an answer that is not the whole set of matches: the hit ceiling was reached, or the document itself was cut while it was built. Read what you found with `part=N`. A `find` is a conversion - it converts what it searches - so it costs and consumes exactly what the same request without `find` would. |
| `capacity` | `low`, `medium`, `high`, `max`                                                   | Working window of the paid LLM stage, symmetric in input and output tokens: `low` (the 16K floor inside the minimum price), `medium`, `high`, or `max` - the whole selected input, and the default. It narrows what is sent to the model, never what is returned: the markdown a caller receives is not cut.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `ttl`      | seconds, honored as named (`0` disables; above a year - 31536000 - clamps to it) | How long the converted markdown may be reused. Reuse is the caller's to ask for: without `ttl` a paid request keeps nothing, the free testing tier keeps a short default, and `ttl=0` stores nothing. A named `ttl` is also the consent to store what `includes=attachments` returns - the images then travel as temporary `GET /att/{id}` links with this lifetime instead of inline data URIs.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `includes` | `attachments`                                                                    | Extra products on top of the answer: `attachments` adds the resource's embedded images where the format carries them (office documents, PDF, EPUB, HTML/RTF, legacy DOC/XLS/PPT), inline as data URIs by default and as `GET /att/{id}` links when `ttl` is set. Each entry carries the image's name, type and size, plus its pixel size, source position and the part it stands in when those are known - enough to decide which image is worth fetching or looking at. The index describes the answer, never the document: only the images whose placeholder stands in the markdown being returned are listed, and only those are resolved - so `part=35` over a 500-page report lists and encodes the pictures of page 35 and leaves the other 199 alone. Without `part` the answer is the whole document and the index is the whole inventory. `find` and `result=meta` answer with the index and no bytes at all: they name which part each image sits in, and `part=N&includes=attachments` then brings the bytes of the one worth having.                                                                                   |

All five protocols expose the same `input` source and apply the same transformations, streaming, and prompt-driven processing - the only
difference is the transport (REST query/JSON, MCP `tools/call`, ACP `session/prompt`, OpenAI `messages`, A2A `message.parts`).

### Asking again without sending the document again

A converted document is addressed by a `resource_id`, and a second question about it carries that address instead of the document. The first request converts and answers; the next one sends `input=mdapi:...` - alone or with a `prompt` - and the document is not uploaded, fetched or converted again.

```
GET /?input=https://example.com/report.pdf
{"success":true,"resource_id":"mdapi:1-<contentHash>", ...}

GET /?input=mdapi:1-<contentHash>&prompt=What+does+it+say+about+revenue
```

| Field / header                        | Meaning                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `resource_id` / `X-MDAPI-Resource-Id` | The address of the converted document this answer is about, in the form `mdapi:<schemaVersion>-<contentHash>`. Send it back as `input` - alone or together with a `prompt` - and the document is answered without being uploaded or fetched again: `input=mdapi:<address>` is read as resource mode, and every other parameter (`prompt`, `result`, `part`, `find`, `capacity`, `includes`) works exactly as it does for any other input. The address is opaque: pass it back unchanged. It is published only when the answer left an entry to address - a paid request names a `ttl` for that, while the free testing tier leaves a default entry - and it lives exactly as long as that entry, so an expired address is refused with `resource_not_found`, indistinguishable from one that never existed. |

An address is a handle to the converted document, not a copy of it and not a link: it does not open in a browser, it is not a listing, and it does not outlive its cache entry. Reading one costs the same as any other request.

### Reading a document in parts

Walking a large document part by part: the range a conversion reports, the headers carrying it, and `part=N`.

The full guide: https://mdapi.io/docs/parts

### Searching a document without reading it

Searching a converted document for literal substrings and getting addresses back instead of text.

The full guide: https://mdapi.io/docs/find

### Documents larger than the input gate

What the input ceiling applies to, and which formats stay readable when a document is larger than the gate.

The full guide: https://mdapi.io/docs/large-documents

### Reading the embedded images

The images a document carries - inline on GET, indexed in `attachments[]` on POST.

The full guide: https://mdapi.io/docs/images

### Where the index rides

The index belongs to the conversion, not to the transport: the core builds `parts`, `find` and `attachments[]` once and every protocol hands them to the client, so nothing is lost by connecting through one endpoint instead of another - only the slot differs (below). A protocol's own shape is never bent to carry them. When `stream` is on, the core sends the index before the content (a `products` frame) and the protocol attaches it to its final frame.

| Protocol | Where the index rides                                                                 |
| -------- | ------------------------------------------------------------------------------------- |
| REST     | The POST JSON body, at the top level                                                  |
| MCP      | The `tools/call` result (JSON text content, declared by `outputSchema`)               |
| A2A      | `artifact.metadata` on the task artifact - in a stream, on the final `artifactUpdate` |
| ACP      | `_meta` on the `PromptResponse` result                                                |
| OpenAI   | The response body, at the top level beside `prompt_result`                            |

## Free tier

A conversion sent without a paid token is drawn from the free allowance, and the counters ride on the answer itself - as headers and as body fields - so a client knows where it stands without asking.

| Field / header                        | Meaning                                                                                                                                                                                                            |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `free_remaining` / `X-Free-Remaining` | How many free conversions are left to this caller today, counted after the one being answered. The free tier carries 10 conversions a day to a caller, drawn from one shared daily allowance for the whole service |
| `free_reset` / `X-Free-Reset`         | Unix timestamp when the free allowance is restored: the next UTC midnight                                                                                                                                          |

The counters travel as `X-Free-Remaining` and `X-Free-Reset` headers on every protocol, and beside them as `free_remaining` and `free_reset` where the answer has a JSON body (POST /, MCP, OpenAI) - so a client reading either one knows where it stands. They are absent on a request answered by a paid token, which is not drawn from the free allowance. The allowance is per caller but drawn from one shared daily pool, so a caller can find it already spent by others. A request the service refuses - an invalid parameter, an unsupported format, a document over the input gate - does not consume it.

### Content warnings

| Field / header                     | Meaning                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `warnings[]` / `X-Content-Warning` | `unrendered-template` - the source is a page whose text is assembled by client-side JavaScript, so the markup that arrived did not carry it. Either it still holds unfilled template placeholders where the text belongs, or it holds almost no text at all because a browser check, a script-built shell or a loading screen stands where the content should be. The conversion is served rather than refused, because an empty answer would be worse, and the warning names the doubt. A client that can render the page itself will get the text the markup stands for |
| `X-Content-Warning`                | `attachments-not-delivered` - the request asked for `includes=attachments` on an answer with no slot for the index: a GET whose `result=prompt` serves the prompt answer alone. The answer arrives as asked and the images are not built at all, rather than built and dropped. Ask for `result=both` or `result=markdown`, turn on `stream`, or use POST, and the images come with the answer. A GET has no JSON body, so this value travels on the header alone                                                                                                         |

A warning is never an error: the conversion is served, and the warning names the doubt beside it.

## Limits

| Limit               | Value                                                                            |
| ------------------- | -------------------------------------------------------------------------------- |
| **Max file size**   | 50 MB                                                                            |
| **Max URL content** | 50 MB                                                                            |
| **Rate limit**      | 10,000 requests per hour                                                         |
| **Free tier**       | 10 requests per day (no token required), within the service's overall free quota |
| **Paid tier**       | min $0.01 per conversion (USDC on Solana)                                        |
| **Token validity**  | 1 year                                                                           |

## Authentication

**Recommended:** Use `Authorization: Bearer TOKEN`

| Method               | Use Case                           |
| -------------------- | ---------------------------------- |
| Bearer (recommended) | `-H "Authorization: Bearer TOKEN"` |
| Header               | `-H "X-Token-Required: TOKEN"`     |

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

| Parameter  | Type    | Required | Description                                                                                                                |
| ---------- | ------- | -------- | -------------------------------------------------------------------------------------------------------------------------- |
| `input`    | string  | *        | Content to convert (URL, text, or data URI - auto-detected)                                                                |
| `prompt`   | string  |          | Custom instructions for LLM processing                                                                                     |
| `result`   | string  |          | Response format: `markdown`, `prompt`, `both`, `meta`                                                                      |
| `stream`   | boolean |          | Enable streaming: true for SSE response                                                                                    |
| `part`     | string  |          | Parts of the document to convert - see below                                                                               |
| `find`     | string  |          | Search the converted document, answer with addresses                                                                       |
| `capacity` | string  |          | Working window of the paid model stage                                                                                     |
| `ttl`      | integer |          | Seconds the converted markdown may be reused; without it a paid request keeps nothing, the free tier keeps a short default |
| `includes` | string  |          | Extra products (e.g. embedded images)                                                                                      |
| `token`    | string  |          | Access token for paid tier                                                                                                 |
| `memo`     | string  |          | Memo for token activation                                                                                                  |

*The `input` parameter is required.*

> **⚠️ Browser URL limit:** GET requests with long `input` or `prompt` values may exceed browser URL limits (~2048 characters). Use POST with JSON body for large payloads.

> **URLs that carry their own query string (`&`).** A raw `&` splits the GET query string: `?input=https://host/api?sql=A&id=2` delivers `input=https://host/api?sql=A` plus a stray `id=2`. The service rebuilds the missing part - every GET parameter it does not define itself is treated as a parameter of the target resource and appended back to `input` in the order you sent it, before the fetch. This is a repair of a truncated request, not a substitution: the robust way is to percent-encode the whole `input` value (as in the example below) or to use POST with a JSON body, where `&` is not special.

### POST / (Content conversion via JSON)

Supports content conversion via JSON body. The `input` parameter accepts URLs, text, or data URIs (auto-detected). Returns a JSON object containing the Markdown content.

#### Parameters

| Parameter  | Type    | Required | Description                                                                                                                |
| ---------- | ------- | -------- | -------------------------------------------------------------------------------------------------------------------------- |
| `input`    | string  | *        | Content to convert (URL, text, or data URI - auto-detected)                                                                |
| `prompt`   | string  |          | Custom instructions for LLM processing                                                                                     |
| `result`   | string  |          | Response format: `markdown`, `prompt`, `both`, `meta`                                                                      |
| `stream`   | boolean |          | Enable streaming: true for SSE response                                                                                    |
| `part`     | string  |          | Parts of the document to convert - see below                                                                               |
| `find`     | string  |          | Search the converted document, answer with addresses                                                                       |
| `capacity` | string  |          | Working window of the paid model stage                                                                                     |
| `ttl`      | integer |          | Seconds the converted markdown may be reused; without it a paid request keeps nothing, the free tier keeps a short default |
| `includes` | string  |          | Extra products (e.g. embedded images)                                                                                      |
| `token`    | string  |          | Access token for paid tier                                                                                                 |
| `memo`     | string  |          | Memo for token activation                                                                                                  |

*The `input` parameter is required.*

### Result Format Parameter

The `result` parameter controls the response format for both GET and POST requests.

| Value                | Description                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| `markdown` (default) | Returns the converted Markdown content                                                         |
| `prompt`             | Returns the result of LLM processing with `prompt` instructions                                |
| `both`               | Returns both `markdown` and `prompt_result` in the response                                    |
| `meta`               | Reconnaissance only: the parts of the document and their sizes, without converting the content |

**Response body by method.** `GET` always answers with Markdown (`Content-Type: text/markdown`) - errors and `402` included. `POST` always answers with the JSON envelope. Source content that has no Markdown form of its own (JSON, JavaScript, source code, configuration, tab-separated data) is delivered verbatim inside a fenced code block, so its own syntax is never read as Markdown.

When `result=both`:

- **GET requests** return Markdown combining `markdown`, followed by "## Prompt Result" and `prompt_result` (always in Markdown format)
- **POST requests** return JSON with `markdown` and `prompt_result` fields

> **Auto `result`:** When `prompt` is provided without an explicit `result`, the core automatically sets `result="prompt"` (LLM output only). Without `prompt`, default is `result="markdown"`. Only specify `result` explicitly when you need both (`result="both"`).

### Prompt Parameter

The `prompt` parameter lets you specify custom instructions for the LLM to follow when generating the result.

| Use Case           | Example                                                    |
| ------------------ | ---------------------------------------------------------- |
| Summarize          | `?input=https://example.com&prompt=Summarize`              |
| Extract key points | `?input=Hello World&prompt=Extract key points`             |
| Convert to JSON    | `?input=https://example.com&prompt=Convert to JSON format` |
| Analyze content    | `?input=Hello World&prompt=Analyze and explain`            |

### Streaming Parameter

Server-Sent Events delivery: the `stream` parameter, the frame shape and the termination markers.

The full guide: https://mdapi.io/docs/streaming

### Response Codes

| Code | Description            | Response Body (GET)                                                                                                                                       | Response Body (POST)                                                                                                                                                        |
| ---- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200  | Success                | Markdown content                                                                                                                                          | JSON with `success`, markdown, prompt_result, metrics, token fields                                                                                                         |
| 402  | Payment Required       | Markdown payment instructions (`X-Error-Code: payment_required`)                                                                                          | JSON with `success:false`, `code:"payment_required"`, and payment object                                                                                                    |
| 400  | Bad Request            | Markdown error + `X-Error-Code` header                                                                                                                    | JSON `{"success":false,"error":"...","code":"invalid_request"}`                                                                                                             |
| 401  | Unauthorized           | Markdown error + `X-Error-Code` header                                                                                                                    | JSON `{"success":false,"error":"...","code":"unauthorized"}`                                                                                                                |
| 404  | Not Found              | Markdown error + `X-Error-Code` header                                                                                                                    | JSON `{"success":false,"error":"...","code":"not_found"}`                                                                                                                   |
| 405  | Method Not Allowed     | POST-only: `/v1/chat/completions` and `/v1/responses` answer a GET with 405 (`/acp` and `/a2a` answer a GET with 404 - a GET of theirs is not a resource) | JSON `{"success":false,"error":"...","code":"method_not_allowed"}`                                                                                                          |
| 413  | Payload Too Large      | Markdown error + `X-Error-Code` header                                                                                                                    | JSON `{"success":false,"error":"...","code":"too_large"}`                                                                                                                   |
| 415  | Unsupported Media Type | Markdown error + `X-Error-Code` header                                                                                                                    | JSON `{"success":false,"error":"...","code":"unsupported_format"}` - or `"code":"encrypted_archive"` for a password-protected archive                                       |
| 429  | Rate Limited           | Markdown error + `X-Error-Code` header                                                                                                                    | JSON `{"success":false,"error":"...","code":"rate_limited"}`                                                                                                                |
| 500  | Server Error           | Markdown error + `X-Error-Code` header                                                                                                                    | JSON `{"success":false,"error":"...","code":"server_error"}`                                                                                                                |
| 502  | Source Unavailable     | Markdown error + `X-Error-Code` header                                                                                                                    | JSON `{"success":false,"error":"...","code":"source_unavailable"}` plus a `source` object naming the status the source itself answered with; nothing was charged            |
| 503  | AI Unavailable         | Markdown error + `X-Error-Code` header                                                                                                                    | JSON `{"success":false,"error":"...","code":"ai_unavailable"}` - the AI provider failed after every channel; nothing was charged, so the same request can simply be retried |

Every error carries a stable `code` string next to the HTTP status - in the JSON body as `code`, and in the `X-Error-Code` header for the plain-markdown responses. **Branch on the code, never on the message text**; the codes are durable, the messages are not.

### Source errors

When the URL you asked for could not be fetched, the answer is `502` with `code: "source_unavailable"`, and a `source` object says what the far side did - so the decision to try again does not have to be read out of an English sentence.

| Field / header                            | Meaning                                                                                                                                                                                                                                                                                       |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `source.url`                              | The address the service actually fetched, after any redirects. It is the caller's own URL unless the source redirected it, and it names what the failure is about                                                                                                                             |
| `source.status` / `X-MDAPI-Source-Status` | The HTTP status the source answered with, as a number. `null` when the source never answered at all (timeout or network failure)                                                                                                                                                              |
| `source.status_text`                      | The reason phrase that came with that status, exactly as the source wrote it (`Not Found`). Absent when there was no answer                                                                                                                                                                   |
| `source.reason`                           | Which failure this was: `source_answered` (the source replied with a non-success status), `timeout` (no answer within the fetch timeout) or `network_error` (the connection failed)                                                                                                           |
| `source.retryable` / `X-MDAPI-Retryable`  | Whether asking again for this same URL can succeed. `false` means the source has given its verdict and a repeat gets the same answer - a different address or a different document is needed. `true` means the source was temporarily unable or unwilling and the same request may work later |

The service answers `502` when the URL it was asked to convert could not be fetched, and `source` says what the far side did. The status stays the service's own: a source that answers `404` does not make the response a `404`, because the service's endpoint was reached and did its work - the failure is on the far side, which is what `502` means. `source.retryable` follows a fixed rule - `false` for a 4xx the source will repeat (a wrong or gone address, a refusal), `true` for a 5xx, for the 4xx that ask for a pause (408, 425, 429) and for a source that never answered. The body of the source's own error page is deliberately not passed through: it is untrusted third-party content and does not belong in a caller's context. Nothing is charged for this failure.

### Response metadata

The facts an answer carries in its headers: the service that replied, the document validators and the integrity field.

The full guide: https://mdapi.io/docs/response-metadata

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

| Method | Path                                                        | Description                        |
| ------ | ----------------------------------------------------------- | ---------------------------------- |
| GET    | /about                                                      | About service                      |
| GET    | /                                                           | API documentation and conversion   |
| POST   | /                                                           | Convert content (JSON body)        |
| POST   | /v1/chat/completions                                        | OpenAI-compatible chat completions |
| POST   | /v1/responses                                               | OpenAI-compatible Responses API    |
| GET    | /mcp                                                        | MCP server manifest                |
| POST   | /mcp                                                        | MCP RPC endpoint                   |
| POST   | /acp                                                        | ACP RPC endpoint (IDE agents)      |
| POST   | /a2a                                                        | A2A RPC endpoint (agent2agent)     |
| GET    | /health                                                     | Health check                       |
| GET    | /llms.txt                                                   | Documentation index                |
| GET    | /llms-full.txt                                              | Full API documentation             |
| GET    | /.well-known/ai-discovery.json or /ai-discovery.json        | AI discovery                       |
| GET    | /.well-known/agent.json or /agent.json                      | AI Agent discovery                 |
| GET    | /.well-known/agent-card.json or /agent-card.json            | A2A Agent card                     |
| GET    | /.well-known/acp.json or /acp.json                          | ACP manifest                       |
| GET    | /.well-known/x402.json or /x402.json                        | x402 payment manifest              |
| GET    | /.well-known/openapi.json or /openapi.json                  | OpenAPI specification (JSON)       |
| GET    | /.well-known/openapi.yaml or /openapi.yaml                  | OpenAPI specification (YAML)       |
| GET    | /.well-known/mapi.md or /mapi.md                            | MAPI specification                 |
| GET    | /.well-known/skill.md or /skill.md                          | Skill specification                |
| GET    | /.well-known/skills/index.json                              | Legacy skills index                |
| GET    | /.well-known/agent-skills/index.json                        | Agent Skills discovery index       |
| GET    | /.well-known/api-catalog                                    | API catalog                        |
| GET    | /.well-known/mcp/server-cards.json                          | MCP server cards index             |
| GET    | /.well-known/plugin/plugin.json or /.well-known/plugin.json | Agent Plugins manifest             |
| GET    | /.well-known/plugin/mcp.json                                | Agent Plugins MCP config           |
| GET    | /.well-known/plugin/skills/mdapi-conversion/SKILL.md        | Agent Plugins conversion skill     |
| GET    | /docs                                                       | API documentation pages            |

#### Examples

```bash
# URL conversion via GET (free)
curl "https://mdapi.io/?input=https://example.com"

# URL with prompt and result=both (returns markdown + prompt_result)
curl "https://mdapi.io/?input=https://example.com&prompt=Summarize&result=both"

# Text with prompt (auto result=prompt)
curl "https://mdapi.io/?input=Hello World&prompt=Summarize"

# URL that carries its own query string (percent-encode the whole input)
curl --get --data-urlencode "input=https://example.com/api/3/action/datastore_search_sql?sql=SELECT%201&resource_id=abc" "https://mdapi.io/"

# Token activation via GET (activate and use)
curl -H "Authorization: Bearer YOUR_TOKEN" -H "X-Memo-Required: YOUR_MEMO" "https://mdapi.io/?input=https://example.com"

# Paid request with token via GET (using token)
curl -H "Authorization: Bearer YOUR_TOKEN" "https://mdapi.io/?input=https://example.com"

# URL conversion via POST (free)
curl -X POST -H "Content-Type: application/json" -d '{"input":"https://example.com"}' "https://mdapi.io/"

# Text with prompt via POST
curl -X POST -H "Content-Type: application/json" -d '{"input":"Hello World","prompt":"Summarize","result":"both"}' "https://mdapi.io/"

# File upload via POST (data URI)
curl -X POST -H "Content-Type: application/json" -d '{"input":"data:text/plain;base64,SGVsbG8gV29ybGQ="}' "https://mdapi.io/"

# Token activation via POST
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer YOUR_TOKEN" -H "X-Memo-Required: YOUR_MEMO" -d '{"input":"https://example.com"}' "https://mdapi.io/"

# Paid request with token via POST
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer YOUR_TOKEN" -d '{"input":"https://example.com"}' "https://mdapi.io/"

```

### OpenAI Compatible Endpoint

The `/v1/chat/completions` endpoint provides an OpenAI-compatible API for markdown conversion with streaming support.

**Supported features:**
- URL extraction from message content (any text containing https?://)
- image_url in messages (OpenAI format) - supports HTTP URLs and data URLs
- file in messages (OpenAI format) - base64 encoded files (field `file.data`, optional `file.mimeType`; built into a `data:` URI for the core - no `file.filename` required or used)
- Direct text content in messages (any text without a URL is sent to the core as the `input` source and converted to Markdown)
- Token and memo via headers (recommended for POST)
- Streaming SSE responses (`stream: true`)
- Custom instructions with LLM processing (system messages, or user messages containing instruction keywords such as *extract, summarize, analyze, format, convert to, write as, create, generate, json* → LLM-driven summary/extraction/transformation)
- `prompt` for LLM-processed output. The response surfaces `prompt_result` at the top level alongside the standard `choices[].message.content` (which carries `prompt_result` when prompt is set, otherwise the Markdown).

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
      "enum": ["markdown", "prompt", "both", "meta"],
      "description": "Response format when using prompt. `meta` answers with the index alone - parts, sizes, metadata - and no content"
    },
    "part": {
      "type": "string",
      "description": "Read a part instead of the whole document: a number, a range (`N-M`), or a comma-separated list"
    },
    "find": {
      "type": "string",
      "description": "Search the converted document and answer with the addresses of the matches - part and offset - never the matched text"
    },
    "capacity": {
      "type": "string",
      "enum": ["low", "medium", "high", "max"],
      "description": "Working window of the paid LLM stage, symmetric in input and output tokens"
    },
    "ttl": {
      "type": "integer",
      "description": "Seconds the converted markdown may be reused; `0` disables reuse"
    },
    "includes": {
      "type": "string",
      "description": "Extra products on top of the answer; `attachments` adds the resource's embedded images"
    },
    "input": {
      "type": "string",
      "description": "Content to convert (URL, text, or data URI - auto-detected). Alternative to a URL/file embedded in messages"
    }
  },
  "required": ["messages"]
}
```

> **Token and memo never travel in the body:** this endpoint reads them from the request - `Authorization: Bearer TOKEN` and `X-Memo-Required: MEMO`, or the `token` and `memo` query parameters.

> **Note on streaming + custom instructions:** when custom instructions trigger LLM processing, the response is returned as a single completion (streaming is not applied to the LLM pass). Streaming SSE applies to the standard conversion path.

### OpenAI Responses Endpoint

`POST /v1/responses` speaks the OpenAI Responses API against the same conversion core. `input` takes a string or an array of items, and `instructions` becomes the system message.

| Part or field               | Meaning                                                                                                                                                                                                                                                                                 |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `input`                     | A string, or an array of items. A bare string, or an item whose `content` is a string, is read as one message; an item whose `content` is an array carries parts. `instructions` becomes the system message                                                                             |
| `input_text` / `text`       | The text of a part                                                                                                                                                                                                                                                                      |
| `input_file`                | One file per request. `file_data` is a data URI (`data:<mime>;base64,...`); `file_url` is an http(s) URL or a data URI. `file_id` on its own is refused with an explanation, because the service is stateless and has no Files API - alongside inline content it is accepted as a label |
| `input_image` / `image_url` | An image part, `image_url` being a string or an object with `url`. A `data:` URI carries the bytes, an http(s) URL is fetched                                                                                                                                                           |

The answer is a `response` object - `{id, object: "response", output: [...], usage: {...}}` - beside the core's own products (`parts`, `find`, `attachments[]`). With `stream: true` the stream carries the Responses event names in order: `response.created`, `response.in_progress`, `response.output_item.added`, `response.content_part.added`, `response.output_text.delta`, `response.output_text.done`, `response.content_part.done`, `response.output_item.done`, `response.completed`.

## MCP Configuration

Connect mdapi.io to your MCP-compatible client (spec 2026-07-28, stateless).

> **Single source via `input`:** the `convert` tool accepts a unified `input` parameter - the same source as every other protocol. The core auto-detects whether the value is a URL, data URI, or text.
> See [Source Parameters (all protocols)](#source-parameters-all-protocols).

### Protocol Requirements

- **Transport:** Streamable HTTP (POST-only for JSON-RPC, GET for manifest)
- **Required headers:** `MCP-Protocol-Version: 2026-07-28` and `Mcp-Method` on every request; `Mcp-Name` additionally on `tools/call`, `resources/read`, and `prompts/get`
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

> **Note:** No token is required to connect. A free tier is available (10 requests per day), within the service's overall free quota.

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

**Activation** - include `token` + `memo` in the first request:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "input": "https://example.com",
      "token": "YOUR_TOKEN",
      "memo": "YOUR_PAYMENT_MEMO"
    }
  }
}
```

**After activation** - use `token` only (no memo needed):

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "convert",
    "arguments": {
      "input": "https://example.com",
      "token": "YOUR_ACTIVATED_TOKEN"
    }
  }
}
```

### MCP Tool Examples

The `convert` tool called over MCP: JSON-RPC requests with a prompt and a result format.

The full guide: https://mdapi.io/docs/examples/mcp

### Using Environment Variables

Or use environment variable:

```bash
export MDAPI_TOKEN=YOUR_ACTIVATED_TOKEN
```

## Code Examples

### JavaScript (fetch)

The conversion called from JavaScript with `fetch` - GET answers Markdown, POST answers JSON with metadata.

Full example: https://mdapi.io/docs/examples/javascript

### Python

The same conversion called from Python.

Full example: https://mdapi.io/docs/examples/python

### Go

The same conversion called from Go.

Full example: https://mdapi.io/docs/examples/go

### Rust

The same conversion called from Rust.

Full example: https://mdapi.io/docs/examples/rust

### OpenAI SDK

The conversion called through the OpenAI-compatible client, streaming included.

Full example: https://mdapi.io/docs/examples/openai

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
  "method": "SendMessage",
  "params": {
      "message": {
        "messageId": "msg-uuid-1",
         "parts": [
           { "text": "https://example.com" }
        ]
      }
  }
}
```

### A2A Methods

| Method               | Description                                                   |
| -------------------- | ------------------------------------------------------------- |
| SendMessage          | Send a message to initiate conversion                         |
| SendStreamingMessage | Send message with SSE streaming updates                       |
| GetTask              | Get task status and results by ID                             |
| ListTasks            | List the tasks of one conversation, one page at a time        |
| CancelTask           | Cancel an in-progress task                                    |
| SubscribeToTask      | Subscribe to updates of a task that is still running, via SSE |

> **Single source via `input`:** the `input` parameter in the message parts is the unified source - the same as the REST endpoint. A bare URL
> inside a text part (e.g. `"https://example.com"`) is extracted automatically and used as the conversion source, so you don't need to wrap it
> in structured JSON. Instructions such as `Summarize` should be passed via the structured `{ "input": "...", "prompt": "..." }` form, not mixed into the text.

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
          { "text": "https://example.com" }
        ]
      }
    }
  }'
```

#### SendMessage with file (data URI)

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "SendMessage",
    "params": {
      "message": {
        "messageId": "msg-uuid-2",
        "parts": [
          { "text": "{"input":"data:text/plain;base64,SGVsbG8gV29ybGQ="}" }
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
              "input": "https://example.com",
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
    "method": "SendMessage",
    "params": {
      "message": {
        "messageId": "msg-uuid-7",
        "parts": [
          {
            "data": {
              "input": "https://example.com",
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

`contextId` is what makes a second call a follow-up rather than a new conversation. Send the one
`SendMessage` answered with, or send your own on the first call - a context id you chose is kept as it
is, so a group of callers can name the thread before any task exists and every later task joins it. The
same id is what `ListTasks` is scoped by.

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
          {
            "text": "{"input":"https://example.com","prompt":"Convert the tables to JSON"}"
          }
        ]
      }
    }
  }'
```

#### GetTask

Read one task by its id:

| Parameter       | Meaning                                                                                                                                                                                                                            |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`            | The task id from `SendMessage`, `SendStreamingMessage` or `ListTasks`. A task is read by its id alone: the id is what grants access, so no other credential is asked for and none is kept with the task.                           |
| `historyLength` | How much of the task's message history to return: not set - the stored history; `0` - the `history` field is left out; a positive number - at most that many most recent messages. A negative value is rejected as invalid params. |
| `tenant`        | Accepted only empty. The service is single-tenant, so a named tenant has no meaning here and a non-empty value is rejected as invalid params.                                                                                      |

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 5,
    "method": "GetTask",
    "params": { "id": "task_12345" }
  }'
```

#### ListTasks

List the tasks that belong to one conversation:

| Parameter              | Meaning                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `contextId`            | The conversation to list - the `contextId` returned by `SendMessage`, or one the caller chose and sent with it: a `SendMessage` that carries its own `contextId` keeps it, so a swarm can name the thread before the first task exists and every later task joins it. A listing is always scoped to one context: tasks are not filed under an account, so the context is what says which tasks belong together. Without it the answer is an empty list, never the whole store. |
| `pageSize`             | How many tasks one page carries. Defaults to 50, minimum 1, maximum 100; a value outside that range is rejected as invalid params. The size actually used comes back as `pageSize`.                                                                                                                                                                                                                                                                                            |
| `pageToken`            | The cursor from the previous page's `nextPageToken`. Opaque - pass it back unchanged. Absent or empty starts at the first page.                                                                                                                                                                                                                                                                                                                                                |
| `status`               | Return only tasks in this state (`TASK_STATE_WORKING`, `TASK_STATE_COMPLETED`, `TASK_STATE_FAILED`, `TASK_STATE_CANCELED`, `TASK_STATE_REJECTED`). Any other value is rejected as invalid params.                                                                                                                                                                                                                                                                              |
| `statusTimestampAfter` | Return only tasks whose status was updated after this ISO 8601 timestamp - the way to ask for what changed since the previous call.                                                                                                                                                                                                                                                                                                                                            |
| `historyLength`        | How much of each task's message history to carry, with the same value semantics as in `GetTask`: a positive number - at most that many most recent messages; `0` - the `history` field is left out. A listing carries no history unless it is asked for, so the default (not set) leaves `history` out as well. A negative value is rejected as invalid params. Credentials are never part of a stored history, so nothing secret is returned by asking for it.                |
| `includeArtifacts`     | Whether each task carries its artifacts. Defaults to `false` to keep a page small; unless it is `true` the `artifacts` field is left out of every task rather than returned empty.                                                                                                                                                                                                                                                                                             |
| `tenant`               | Accepted only empty. The service is single-tenant, so a named tenant has no meaning here and a non-empty value is rejected as invalid params instead of being answered with tasks that are not the caller's.                                                                                                                                                                                                                                                                   |

A page carries at most `pageSize` tasks, most recently updated first. The result names the page size
that was used and how many tasks the whole query matches:

| Field           | Meaning                                                                                                                    |
| --------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `tasks`         | The page of tasks, most recent status first, carrying `artifacts` and `history` only as far as the request asked for them. |
| `nextPageToken` | Cursor for the following page; empty means this was the last page.                                                         |
| `pageSize`      | The page size the service actually used - the requested value, or the default 50.                                          |
| `totalSize`     | How many tasks match the whole query, before pagination.                                                                   |

Walk a whole conversation by passing `nextPageToken` back as `pageToken` until it comes back empty:

```bash
# First page
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 6,
    "method": "ListTasks",
    "params": { "contextId": "ctx_12345", "pageSize": 2 }
  }'

# Result: {"tasks":[{...},{...}],"nextPageToken":"eyJ0IjoiMjAyNi0wNi0yNlQxNjowMDowMC4wMDBaIiwiaSI6InRhc2tfMTIzNDUifQ","pageSize":2,"totalSize":5}

# Next page - the token is opaque, pass it back unchanged
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -d '{
    "jsonrpc": "2.0",
    "id": 7,
    "method": "ListTasks",
    "params": { "contextId": "ctx_12345", "pageSize": 2, "pageToken": "eyJ0IjoiMjAyNi0wNi0yNlQxNjowMDowMC4wMDBaIiwiaSI6InRhc2tfMTIzNDUifQ" }
  }'
```

**Reading a task is what returns its content.** A listing is an inventory: it carries no history unless
`historyLength` asks for it, and no artifacts unless `includeArtifacts` is `true`, so one call over a
long conversation stays small. The result of a conversion is read with `GetTask`, one task
per call.

**Task records are bounded and temporary.** A record keeps the caller's message parts up to
65536 characters each - a file part keeps its `mediaType` but not its bytes -
and credentials (`token`, `memo`) are never written into a task. A record has a size ceiling, and what
does not fit is left out in a fixed order, with the record saying which of the two happened. The caller's
own message goes first, since they still hold it, and then the tail of the result: what stays is the
beginning of it, marked `metadata.contentTruncated: true`, so a large result is still readable in part
rather than missing. The response to `SendMessage` is never affected - it always carries the whole
result. `metadata.contentStored: false` is the stronger statement, and it means the result is not in the
record at all. A task that has finished stays readable for about
5 minutes and a task still running for up to
60 minutes, after which `GetTask` answers `-32001`.

### A2A Message Parts

Messages use the A2A `Part` format (field-name discriminators per spec v1.0.0):

| Type   | Description                                                                                           | Fields                                  |
| ------ | ----------------------------------------------------------------------------------------------------- | --------------------------------------- |
| `text` | Plain text content or JSON-encoded params                                                             | `text`                                  |
| `raw`  | File content as base64 bytes; normalized to a data URI (`data:<mediaType>;base64,<raw>`) for the core | `raw` (base64), `mediaType` (optional)  |
| `data` | Structured JSON data (core params)                                                                    | `data` (object), `mediaType` (optional) |
| `url`  | URL to fetch and convert                                                                              | `url` (http/https)                      |

**Part → Core Parameter Mapping:**
- `text` Part → `input` param (direct content) or JSON-encoded params (`{ "input": "...", "prompt": "..." }`)
- `raw` Part → `input` param as a data URI (`data:<mediaType>;base64,<raw>`), decoded by the core
- `data` Part → merged as params (`input`, `prompt`, `result`, `part`, `find`, `capacity`, `ttl`, `includes`, `stream`, `token`, `memo`)
- `url` Part → `input` param (fetched and converted)

### Message Object

```typescript
interface Message {
  messageId: string;                  // REQUIRED: unique ID (e.g. "msg-uuid")
  contextId?: string;                 // Optional: the conversation to join; send your own to name it, omit for a generated one
  taskId?: string;                    // Optional: associate with existing task
  role: "user" | "agent";   // REQUIRED
  parts: Array<Part>;                 // REQUIRED: at least one part
}
```

### Task Data Model

```typescript
interface Task {
  id: string;                         // "task_<uuid>"
  contextId: string;                  // "ctx_<uuid>", or the one the caller sent with SendMessage
  status: {
    state: string;                    // "TASK_STATE_WORKING" | "TASK_STATE_COMPLETED" | "TASK_STATE_FAILED" | "TASK_STATE_CANCELED" | "TASK_STATE_REJECTED"
    timestamp: string;                // ISO 8601
    message?: Message;                // only on failure
  };
  artifacts?: Array<{
    artifactId: string;
    name: string;
    metadata?: {                     // the core's products: parts, find, attachments[], warnings[]
      parts?: object;
      find?: object;
      attachments?: Array<object>;
      warnings?: Array<string>;
    };
    parts: Array<Part>;
  }>;
  history?: Array<Message>;
  metadata?: {
    historyStored?: boolean;         // false: the caller's own message did not fit the record
    contentStored?: boolean;         // false: the result is not in the record at all
    contentTruncated?: boolean;      // true: what is stored is the beginning of the result
  };
}
```

### Task States

| State                  | Description                   |
| ---------------------- | ----------------------------- |
| `TASK_STATE_WORKING`   | Task is being processed       |
| `TASK_STATE_COMPLETED` | Task finished successfully    |
| `TASK_STATE_FAILED`    | Task failed during processing |
| `TASK_STATE_CANCELED`  | Task was canceled by client   |
| `TASK_STATE_REJECTED`  | Task was rejected by server   |

### Error Responses

A2A uses JSON-RPC 2.0 error format with A2A-specific error codes:

| Code     | Error                             | HTTP | Description                                                |
| -------- | --------------------------------- | ---- | ---------------------------------------------------------- |
| `-32700` | Parse error                       | 400  | Invalid JSON payload                                       |
| `-32600` | Invalid Request                   | 400  | Missing required fields (message.parts, message.messageId) |
| `-32601` | Method not found                  | 400  | Unknown A2A method                                         |
| `-32602` | Invalid params                    | 400  | A parameter is out of range or not accepted                |
| `-32603` | Internal error                    | 500  | The request could not be completed                         |
| `-32001` | Task not found                    | 404  | Task ID does not exist                                     |
| `-32002` | Task is not in a cancelable state | 400  | CancelTask on a task that already finished                 |
| `-32004` | UnsupportedOperationError         | 400  | SubscribeToTask on a task that already finished            |

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
           { "text": "https://example.com" }
         ]
      }
    }
  }'
```

Response format (A2A v1.0.0 streaming sequence - exact frames the service emits):

```
data: {"jsonrpc":"2.0","id":5,"result":{"task":{"id":"task_...","contextId":"ctx_...","status":{"state":"TASK_STATE_WORKING","timestamp":"..."},"artifacts":[]}}}
data: {"jsonrpc":"2.0","id":5,"result":{"artifactUpdate":{"taskId":"task_...","contextId":"ctx_...","artifact":{"artifactId":"artifact_...","name":"conversion_result","parts":[{"text":" partial "}]},"append":true,"lastChunk":false}}}
data: {"jsonrpc":"2.0","id":5,"result":{"artifactUpdate":{"taskId":"task_...","contextId":"ctx_...","artifact":{"artifactId":"artifact_...","name":"conversion_result","parts":[{"text":" more "}],"metadata":{"parts":{"count":12,"unit":"page"}}},"append":true,"lastChunk":true}}}
data: {"jsonrpc":"2.0","id":5,"result":{"statusUpdate":{"taskId":"task_...","contextId":"ctx_...","status":{"state":"TASK_STATE_COMPLETED","timestamp":"..."}}}}
data: [DONE]
```

The first frame carries the full `task` in TASK_STATE_WORKING; content streams as
`artifactUpdate` frames (`lastChunk: true` on the final chunk); `statusUpdate` closes the stream with
the terminal TASK_STATE_COMPLETED state. The final `artifactUpdate` carries the core's products
(`parts`, `find`, `attachments[]`, `warnings[]`) in `artifact.metadata`. The persisted task (via `GetTask`) carries the real
`artifacts[].parts[]` content, unless the result is larger than a task record holds (see *List Tasks*).

### Subscribe to Task

Subscribe to a task that is still running, for real-time updates:

```bash
curl -X POST https://mdapi.io/a2a   -H "Content-Type: application/a2a+json"   -H "Accept: text/event-stream"   -d '{
    "jsonrpc": "2.0",
    "id": 6,
    "method": "SubscribeToTask",
    "params": { "id": "task_12345" }
  }'
```

A task that has already finished cannot be subscribed to: the call answers `-32004`
`UnsupportedOperationError` (HTTP 400), and the finished task is read with `GetTask`.

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

## ACP Configuration

Connect mdapi.io to your IDE or coding agent (JetBrains, Cursor, VS Code, etc.) via the Agent Client Protocol v1.0.0. ACP is a JSON-RPC 2.0 endpoint at `POST /acp`. Its native surface is **session/turn**: create an ephemeral session, then send a prompt - the converted content streams back as `session/update` notifications.

> **Single source via `input`:** each `session/prompt` carries a unified `input` - the same source as every other protocol. The core auto-detects whether the value is a URL, data URI, or text.
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

Or call the JSON-RPC endpoint directly with `POST /acp` (Content-Type: `application/json`). Sessions are **stateless and ephemeral** - no session state is kept server-side, so each `session/prompt` runs as an independent conversion:

```bash
# 1. Create a session
curl -X POST https://mdapi.io/acp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"session/new","params":{}}'

# 2. Send a prompt (content streams back as session/update notifications)
curl -X POST https://mdapi.io/acp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":2,"method":"session/prompt","params":{
    "sessionId":"<sessionId from session/new>",
    "prompt":[{"type":"text","text":"<instructions>"},
              {"type":"resource_link","uri":"https://example.com"}]
  }}'
```

> **Note:** `GET /acp` is not supported (ACP is POST-only). A free tier is available without a token.

### ACP Methods

| Method           | Description                                                                     |
| ---------------- | ------------------------------------------------------------------------------- |
| `initialize`     | Handshake: return protocol version, agent capabilities, and agent info          |
| `session/new`    | Create an ephemeral, stateless session (returns a `sessionId`)                  |
| `session/prompt` | Run a conversion turn; content streams back as `session/update` notifications   |
| `session/cancel` | Notification (204, no response body) that best-effort cancels an in-flight turn |

### ACP Session Examples

**URL conversion (instructions + `resource_link`):**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "session/prompt",
  "params": {
    "sessionId": "<sessionId>",
    "prompt": [
      { "type": "text", "text": "Summarize" },
      { "type": "resource_link", "uri": "https://example.com" }
    ]
  }
}
```

**Text conversion (a bare `text` block becomes the input):**

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "session/prompt",
  "params": {
    "sessionId": "<sessionId>",
    "prompt": [{ "type": "text", "text": "Hello World" }]
  }
}
```

**File conversion (a `data:` URI in a `text` block is auto-detected as a file):**

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "session/prompt",
  "params": {
    "sessionId": "<sessionId>",
    "prompt": [
      { "type": "text", "text": "Extract the title" },
      { "type": "text", "text": "data:text/plain;base64,SGVsbG8gV29ybGQ=" }
    ]
  }
}
```

**Token activation (first request only):** every protocol passes the token per-call. Include it on any method's `params` as a `token` (+ `memo` on first use) - ACP does not use a session-level authenticate exchange. The core's other parameters (`part`, `find`, `capacity`, `ttl`, `includes`, `stream`) ride in the same `params` object.

### ACP Response Format

Content is delivered **exclusively via `session/update` notifications**; the final `session/prompt` result carries only the `stopReason` (`PromptResponse`). A successful turn looks like this SSE stream:

```
data: {"jsonrpc":"2.0","method":"session/update","params":{"sessionId":"<sessionId>","update":[{"content":{"type":"text","text":"<converted content>"},"messageId":"<messageId>"}]}}
data: {"jsonrpc":"2.0","id":1,"result":{"stopReason":"end_turn"}}
data: [DONE]
```

- The `session/update` `update[]` array holds the content chunks; `messageId` is stable across all chunks of a single turn.
- The core's products (`parts`, `find`, `attachments[]`, `warnings[]`) ride in the final `PromptResponse` under `_meta`.
- **Streaming:** add `"stream": true` to the `session/prompt` `params` to stream the conversion live - each content chunk arrives as its own `session/update` notification (same `messageId`), then the `PromptResponse`. Without it, the whole turn arrives as one buffered `session/update` notification. Streaming is inherent to ACP; content only ever rides notifications.
- A failed conversion streams an explanatory `session/update`, then ends with `stopReason: "error"`. `session/cancel` is a client→agent notification - the agent acknowledges it with HTTP 204 and no body.
- Token status (`X-Token-Balance`, `X-Token-Expires`, `X-Token-Status`), the free-tier counters (`X-Free-Remaining`, `X-Free-Reset`) and `X-Content-Warning` are proxied from the core into the response headers.

## Usage scenarios

mdapi.io is a minimal, self-documenting service-transport primitive: REST, MCP, ACP, A2A, and OpenAI-compatible endpoints all call the same transformation core, so agents can combine protocols and pass already-processed knowledge between each other.

- **Agent swarms** - each request is handled by a stateless execution environment created for it, so capacity is drawn from the platform as the work arrives rather than reserved ahead of time. An orchestrator fans work out across a swarm of agents, and the swarm converts distinct resources in parallel. No throughput figure is published, because one would be a promise rather than a measurement: the number to plan against is your own budget of 10000 requests/hour counted per caller, not a benchmarked ceiling. Different users may freely access the same resource.
- **Shared vs individual payment** - an orchestrator can pay once for a shared token (batching on-chain activity), or each agent can activate its own token for the exact volume it received.
- **Human-in-the-loop** - if an agent has no wallet or insufficient funds, it returns payment details + a QR code; the human pays from a mobile device and the agent resumes.
- **Role switching** - an agent's role can change mid-task; one agent fetches/normalizes, another summarizes/extracts, relaying compact results via the text or prompt parameters.
- **Cross-protocol interoperability** - knowledge extracted on one protocol is reusable on another.
- **Bulk processing / model training** - the same pattern lets a corpus be converted in parallel instead of one document at a time; what bounds it is the caller's own budget, not a provisioning step.

See https://mdapi.io/about for the full scenario walkthrough.

## Links

- **About service:** https://mdapi.io/about
- **API documentation and conversion:** https://mdapi.io
- **MCP server manifest:** https://mdapi.io/mcp
- **Health check:** https://mdapi.io/health
- **Documentation index:** https://mdapi.io/llms.txt
- **Full API documentation:** https://mdapi.io/llms-full.txt
- **AI discovery:** https://mdapi.io/.well-known/ai-discovery.json or https://mdapi.io/ai-discovery.json
- **AI Agent discovery:** https://mdapi.io/.well-known/agent.json or https://mdapi.io/agent.json
- **A2A Agent card:** https://mdapi.io/.well-known/agent-card.json or https://mdapi.io/agent-card.json
- **ACP manifest:** https://mdapi.io/.well-known/acp.json or https://mdapi.io/acp.json
- **x402 payment manifest:** https://mdapi.io/.well-known/x402.json or https://mdapi.io/x402.json
- **OpenAPI specification (JSON):** https://mdapi.io/.well-known/openapi.json or https://mdapi.io/openapi.json
- **OpenAPI specification (YAML):** https://mdapi.io/.well-known/openapi.yaml or https://mdapi.io/openapi.yaml
- **MAPI specification:** https://mdapi.io/.well-known/mapi.md or https://mdapi.io/mapi.md
- **Skill specification:** https://mdapi.io/.well-known/skill.md or https://mdapi.io/skill.md
- **Agent Plugins manifest:** https://mdapi.io/.well-known/plugin/plugin.json or https://mdapi.io/.well-known/plugin.json
- **Agent Plugins MCP config:** https://mdapi.io/.well-known/plugin/mcp.json
- **Agent Plugins conversion skill:** https://mdapi.io/.well-known/plugin/skills/mdapi-conversion/SKILL.md
- **API documentation pages:** https://mdapi.io/docs

## External Links

- **github.com** https://github.com/mdapiio/mdapi.io
- **skills.sh** https://www.skills.sh/mdapiio/mdapi.io
- **skillsmp.com** https://skillsmp.com/creators/mdapiio/mdapi.io
- **clawhub.ai** https://clawhub.ai/mdapiio
- **x.com** https://x.com/mdapiio

## Disclaimer

**The service is provided "AS IS".**


> mdapi.io is an edge-native service-transport primitive for AI, autonomous-agents, and the Web4 ecosystem.
