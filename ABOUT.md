# About service

mdapi.io - Minimal Data API I/O: a content transformation layer primitive for AI systems.


Transforms files and webpages into AI-ready Markdown and structured data, optimized for LLM efficiency and token usage.

## Principles

1. **One core, five protocols.** REST, MCP, A2A, ACP and the OpenAI-compatible endpoint all call the same conversion core: the transport changes, the result does not.
2. **One description, everywhere.** The documentation and the discovery files are generated from one source, so a fact cannot be true on one surface and stale on another.
3. **Stateless by default.** A request is converted in memory and the environment that ran it is discarded. What can outlive it is a cache entry - a `ttl` on the paid tier, a short default on the free testing tier - and an A2A task record, so that `GetTask` can answer.
4. **Knowledge a model cannot read is still knowledge.** Documents arrive in the formats the world actually uses - PDF, office files, spreadsheets, scans, proprietary binaries - and leave as Markdown or structured data any model can use, with no toolchain on your side.
5. **The caller's resources are the ones being spent.** Extraction and transformation run at the edge and the answer carries only what was asked for - a part, a found offset, a summary - fewer tokens, less context, less compute.
6. **A public document is converted once, not once per caller.** When many callers read the same public resource, a `ttl` lets the work be shared: the conversion is done once and reused while the entry lives, so the same answer costs the energy of one conversion instead of many.

## What mdapi.io does

mdapi.io takes files and webpages and returns exactly what your model or agent needs:

- Markdown - the document's own text and structure, with the markup around it removed.
- Prompt-driven summaries, extractions, or analyses with custom instructions - structured JSON included when you ask for it.
- The conversion and the prompt result together, when you need both.
- Streaming output for large inputs or real-time workflows.

All supported protocols and interfaces ultimately converge on the same transformation core, and the goal is always the same: give you just enough data to be useful, without wasting context or tokens.

## Edge-native design

mdapi.io is built for stateless edge execution.

- Processing happens in memory.
- Nothing to sign up for and nothing to clean up: no account, no document library to maintain - each request stands on its own.
- The service reduces context, token, and infrastructure usage by moving heavy extraction and transformation work to the edge.

It runs on Cloudflare's global edge network rather than on machines sized in advance, and that is where the elasticity described below comes from: each request is executed by an environment created for it, and capacity is drawn from the platform as the work arrives rather than reserved ahead of time. Nothing has to be provisioned, scaled, or warmed up for a burst. What one request may consume - memory, execution time, input size - is bounded per request, and those bounds are the documented ones, not whatever the underlying platform happens to allow.

## Who it is for

### For people

Use mdapi.io when you want to quickly:

- turn a webpage into readable Markdown;
- convert a report or slide deck into plain text you can search and summarize;
- extract key points or structured information from long content;
- prepare cleaner input before sending it to an AI assistant or model.

### For autonomous agents

Use mdapi.io when your agent needs a reliable transformation layer between raw content and reasoning, planning, or downstream action:

- fetch and normalize webpages before analysis;
- convert uploaded files into AI-ready text;
- run prompt-driven summarization or extraction as part of a tool call;
- pass compact results between agents in a multi-step workflow;
- coordinate multi-agent or swarm workflows with compact intermediate outputs;
- use shared or individual payment flows when a task needs paid execution.

Agents can access mdapi.io over REST, MCP, ACP, A2A, or OpenAI-compatible APIs using the same behavior described in the discovery documents.

## Free and paid access

Two tiers, one core. The free testing tier needs no credential and runs everything that is native - every supported format, every protocol, and every parameter that asks no model for an answer - within a daily allowance of 10 conversions for one caller, out of a quota allocated for the whole service, so the full allowance is not guaranteed. The paid tier is opened by a token carrying a prepaid balance (or by an x402 payment that mints one): it adds the stages that cost a model call - `prompt`, `schema` and `ocr` - and charges each conversion from that balance, with no daily allowance. A paid stage asked for on the free tier is never skipped silently: the answer is `402` with the payment terms and the free route for the same input, and nothing is charged. The free tier lets an agent find out whether the service fits the work before paying for it. Paying does not widen the shared budget: the fair-use limit of 10000 requests/hour is counted per caller, and spreading the same work across more agents does not create more of it. Different callers may freely access the same resource.

## Usage scenarios

mdapi.io is a minimal, self-documenting service-transport primitive. Because every protocol converges on the same transformation core, agents can combine them freely - passing already-processed knowledge between each other.

### Agent swarms and parallel processing

An orchestrator can fan work out across a swarm of agents, and the swarm converts many distinct resources in parallel. Each request runs in its own stateless environment, so a swarm adds work without adding coordination, and agents can hand compact intermediate results to one another.

### Shared vs individual payment

- **Shared token** - the orchestrator pays once, activates a token with a prepaid balance, and hands it to the whole swarm; each conversion decrements the balance. This batches on-chain activity and reduces blockchain load.
- **Individual tokens** - each agent pays for its own work, activating exactly the volume it expects to use.
- If a wallet is missing or underfunded, the agent drops into **human-in-the-loop** mode: it returns payment details and a QR code, the human pays from a mobile device, and the agent resumes automatically on confirmation.

### Role switching and multi-agent / human collaboration

An agent's role can change mid-task. One agent may fetch and normalize a webpage, then hand the compact Markdown to another agent (via the input and prompt parameters) that summarizes or extracts. The same flow works between agent and human: an IDE agent indexes scattered project documentation in mixed formats and surfaces sourced snippets; a research agent prepares RAG-ready input; a coordinator delegates and reassembles results.

### Bulk processing and model training

For a task that needs many resources converted quickly (LLM training corpora, large-scale RAG indexing, bulk document normalization), the swarm pattern lets the work be spread across parallel stateless instances instead of run one document at a time. What bounds it in practice is the caller's own budget and the size of each document, not a provisioning step.

### Prompt as a command between agents

The `prompt` parameter is not only instructions for the service - it is also how one agent directs another. An agent can hand a processed resource to a peer together with a prompt like "extract only the price tables" or "return this as JSON", and the receiving agent lets mdapi.io execute that instruction. The service becomes the shared execution engine for the swarm's intent, not just a converter.

### Legacy to AI-ready bridge

Many real systems still speak in scans, spreadsheets, and old document formats. mdapi.io normalizes that mixed, legacy input into clean AI-ready Markdown or structured data on the fly, so agents can treat a decades-old report the same way they treat a modern webpage. It is the translation layer that lets older infrastructure participate in an agentic workflow without refactoring.

### Address before you buy

Conversion is not all-or-nothing. `result=meta` returns the inventory of a document - what it is, how many parts it has, which images it carries - without converting the content itself, so an agent can decide whether the document is worth reading before committing to it. `find` then answers where a term occurs, as part numbers and offsets and never as text, and `part=N` brings that one part. A 500-page report can be explored, located, and then read in the single slice that matters, while the rest never reaches the model.

### A constructor of capabilities

mdapi.io is a global service-transport primitive: each request is assembled from the capabilities you need, brick by brick.

- **Free capabilities.** Converting URLs, files, and text the edge itself can read - documents, webpages, spreadsheets, archives, legacy formats - into Markdown, structure, or inventory, without a paid call.
- **Paid capabilities.** `prompt` (your instruction executed by an LLM), `ocr` (text read out of images), and rendering for pages that build their text in the browser - a paid fallback for a URL the direct fetch could not read. A prepaid token or x402 payment unlocks all of it.
- **Caching is your choice, and retention with it.** The cache is opened by asking for it. A named `ttl` is the caller's own lease: the converted markdown is kept for that lifetime, honored as named up to the one-year ceiling; an identical request inside the window is answered without re-converting it, and reuse never extends it. Without `ttl` a paid request stores nothing at all - the document is converted, the answer returned, and no copy is kept, so a paying caller is never the reason something is held. The free testing tier keeps a short default instead (3600 seconds - one hour; 86400 - twenty-four hours - for the documentation examples) and re-arms it on reuse, so repeated calls over one document cost the service less - that tier is meant for testing, not for production documents. `ttl=0` stores nothing. Reading a resource address is asking to read, and needs no `ttl`. A request that carries a `password` is never cached: it neither reads a stored conversion nor stores one, so a document that was unlocked with a secret is never handed to a caller who did not name the same secret. An address (`resource_id`, `mdapi:...`) exists only in the reuse mode: with nothing stored there is nothing to address. On `POST /a2a` a task record is kept so that `GetTask` and `SubscribeToTask` can answer.

### Why route through the edge at all

Sending raw documents straight into a frontier model works, and for one small page it is often the right call. The trade changes with volume, size, and repetition:

- **Payload.** A webpage or a PDF arrives wrapped in markup, layout and encoding a model cannot use. Converting first means the model reads text instead of a file format - and nothing is dropped on a guess, so what the page says is what the answer carries.
- **Context.** A converted document is something a context window can actually hold; a raw one competes with the conversation it is part of.
- **Cost.** Money as well as resources: reading a document through a conversion costs less than reading it whole through a frontier model.
- **Division of labour.** Extraction and transformation do not need a frontier model. Handing them to one is the most expensive available way to read a file.
- **Uniformity.** One conversion contract across webpages, PDFs, office documents, spreadsheets, archives, and legacy formats means one integration instead of one per format.
- **Addressing.** You can locate before you buy, and read one part of a large document without paying for the rest of it.

The promise is deliberately a narrow one, and it is the one this service can keep: remove weight before reasoning. It is not a substitute for a model, and no conversion makes a poor source good - but cleaner input also means steadier answers, because the model reasons about the text rather than about the format that carries it.

## How to start using mdapi.io

You can integrate mdapi.io in a few different ways:

- Quick tests and scripts - call the REST API directly to convert URLs, text, or files.
- AI tools and frameworks - connect via MCP, ACP, or A2A using the provided manifests and discovery endpoints.
- Existing OpenAI-style clients - point your client to the OpenAI-compatible endpoint and send messages that contain URLs, files, or images.
- Autonomous workflows - use mdapi.io as the transformation step between raw input and reasoning.

If payment is needed, an agent can handle autonomous payment or present a QR code for human approval.

If you need exact parameters, limits, and protocol details, check the documentation and discovery files linked from the root mdapi.io page.

## In one sentence

mdapi.io turns complex input into compact, AI-ready output so humans and autonomous agents can work faster with less noise and fewer tokens.

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
- **x402 payment manifest:** https://mdapi.io/.well-known/x402.json or https://mdapi.io/x402.json or https://mdapi.io/.well-known/x402
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


> mdapi.io is an edge-native service-transport primitive for AI, autonomous agents, and the Web4 ecosystem.
