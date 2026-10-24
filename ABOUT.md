# About service

mdapi.io - Minimal Data API I/O: a content transformation layer primitive for AI systems.


Transforms files and webpages into AI-ready Markdown and structured data, optimized for LLM efficiency and token usage.

## What mdapi.io does

mdapi.io takes files and webpages and returns exactly what your model or agent needs:

- Clean Markdown instead of raw HTML or PDF clutter.
- Structured data when you need fields, tables, or entities.
- Prompt-driven summaries, extractions, or analyses with custom instructions.
- Streaming output for large inputs or real-time workflows.

All supported protocols and interfaces ultimately converge on the same transformation core.

## Edge-native design

mdapi.io is built for stateless edge execution.

- Processing happens in memory.
- Nothing to sign up for and nothing to clean up: no account, no history, no document library to maintain - each request stands on its own.
- The service reduces context, token, and infrastructure usage by moving heavy extraction and transformation work to the edge.

It runs on Cloudflare's global edge network rather than on machines sized in advance, and that is where the elasticity described below comes from: each request is executed by an environment created for it, and capacity is drawn from the platform as the work arrives rather than reserved ahead of time. Nothing has to be provisioned, scaled, or warmed up for a burst. What one request may consume - memory, execution time, input size - is bounded per request, and those bounds are the documented ones, not whatever the underlying platform happens to allow.

## Why use it

Modern AI systems work best with focused, well-structured input. mdapi.io helps you get there by:

- Reducing noise - stripping navigation, boilerplate, and visual layout that models do not need.
- Saving tokens - returning only the essential content, not entire pages or documents.
- Cutting manual work - no need to hand-clean HTML, PDFs, or mixed-format sources.
- Improving output quality - better inputs usually mean more accurate, stable results.
- Simplifying automation - one consistent service for many content types and tools.

Instead of teaching every agent or script how to parse and clean content, you call mdapi.io once and reuse the result wherever needed.

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

## Built for

- IDE and coding agents.
- Research and analysis workflows.
- RAG pipelines and knowledge bases.
- Multi-agent systems and swarms.
- Human-in-the-loop review flows.
- Edge-native AI workflows.
- Protocol-aware service delivery.
- Agent economics and pay-per-use workflows.

## Payment-aware workflows

mdapi.io supports both autonomous and human-assisted payment flows.

- Autonomous agents can use a connected wallet when available for pay-per-use execution.
- Shared swarm budgets can be used for coordinated execution.
- Human users can complete payment manually through QR-based flows when needed.

## Usage scenarios

mdapi.io is a minimal, self-documenting service-transport primitive. Because every protocol converges on the same transformation core, agents can combine them freely - passing already-processed knowledge between each other.

### Agent swarms and parallel processing

Each request is handled by a stateless execution environment created for it, so capacity is drawn from the platform as the work arrives rather than reserved ahead of time. An orchestrator can fan work out across a swarm of agents, and the swarm converts many distinct resources in parallel. Because every protocol converges on the same core, agents can cooperate directly and hand compact intermediate results to one another.

What an agent can plan against is its own budget rather than a platform ceiling. There are two tiers. A free tier of 10 conversions a day exists for one purpose: to let an agent find out whether the service fits the work before paying for it. The paid tier carries a fair-use budget of 10000 requests/hour counted per caller, and paid requests are counted in it. A token is a prepaid balance rather than extra rate - activating one buys volume at $0.01 per conversion and does not widen the budget, and spreading the same work across more agents does not create more of it. Different callers may freely access the same resource.

### Shared vs individual payment

- **Shared token** - the orchestrator pays once, activates a token with a prepaid balance, and hands it to the whole swarm; each conversion decrements the balance. This batches on-chain activity and reduces blockchain load.
- **Individual tokens** - each agent pays for its own work, activating exactly the volume it received (e.g. 166 resources → pay for 166 upfront).
- If a wallet is missing or underfunded, the agent drops into **human-in-the-loop** mode: it returns payment details and a QR code, the human pays from a mobile device, and the agent resumes automatically on confirmation.

### Role switching and multi-agent / human collaboration

An agent's role can change mid-task. One agent may fetch and normalize a webpage, then hand the compact Markdown to another agent (via the input and prompt parameters) that summarizes or extracts. The same flow works between agent and human: an IDE agent indexes scattered project documentation in mixed formats and surfaces sourced snippets; a research agent prepares RAG-ready input; a coordinator delegates and reassembles results.

### Cross-protocol interoperability

Because REST, MCP, ACP, A2A, and OpenAI-compatible endpoints all call the same core, different agent types can cooperate directly. Knowledge extracted on one protocol can be relayed on another - the transport is interchangeable, the result is the same.

### Bulk processing and model training

For a task that needs many resources converted quickly (LLM training corpora, large-scale RAG indexing, bulk document normalization), the swarm pattern lets the work be spread across parallel stateless instances instead of run one document at a time. What bounds it in practice is the caller's own budget and the size of each document, not a provisioning step.

### Edge filter in front of expensive models

The core job of mdapi.io is to remove weight before it reaches reasoning. Instead of sending a raw PDF, webpage, or scanned document straight into an LLM, an agent routes it through mdapi.io first and forwards only the compact result. That keeps context windows small, cuts token spend, and avoids wasting compute on navigation, boilerplate, and layout the model never needed. The service is the heavy-lifting layer; the model gets just the part it must reason about.

### Prompt as a command between agents

The `prompt` parameter is not only instructions for the service - it is also how one agent directs another. An agent can hand a processed resource to a peer together with a prompt like "extract only the price tables" or "return this as JSON", and the receiving agent lets mdapi.io execute that instruction. The service becomes the shared execution engine for the swarm's intent, not just a converter.

### Legacy to AI-ready bridge

Many real systems still speak in scans, spreadsheets, and old document formats. mdapi.io normalizes that mixed, legacy input into clean AI-ready Markdown or structured data on the fly, so agents can treat a decades-old report the same way they treat a modern webpage. It is the translation layer that lets older infrastructure participate in an agentic workflow without refactoring.

### Address before you buy

Conversion is not all-or-nothing. `result=meta` returns the inventory of a document - what it is, how many parts it has, which images it carries - without converting the content itself, so an agent can decide whether the document is worth reading before committing to it. `find` then answers where a term occurs, as part numbers and offsets and never as text, and `part=N` brings that one part. A 500-page report can be explored, located, and then read in the single slice that matters, while the rest never reaches the model.

### A constructor of capabilities

mdapi.io is a global service-transport primitive: each request is assembled from the capabilities you need, brick by brick.

- **Free capabilities.** Converting URLs, files, and text the edge itself can read - documents, webpages, spreadsheets, archives, legacy formats - into Markdown, structure, or inventory, up to 10 conversions a day.
- **Paid capabilities.** `prompt` (your instruction executed by an LLM), `ocr` (text read out of images), and JavaScript rendering - JavaScript executes only in paid conversions. A prepaid token or x402 payment unlocks all of it.
- **Caching is your choice.** Pass a `ttl` (up to a year) and the converted work is cached, so a repeat call returns the stored answer instead of repeating the work. The cache is opened by asking for it. A named `ttl` is the caller's own lease: the converted markdown is kept for that lifetime, honored as named up to the one-year ceiling; an identical request inside the window is answered without re-converting it, and reuse never extends it. Without `ttl` a paid request stores nothing at all - the document is converted, the answer returned, and no copy is kept, so a paying caller is never the reason something is held. The free testing tier keeps a short default instead (3600 seconds - one hour; 86400 - twenty-four hours - for the documentation examples) and re-arms it on reuse, so repeated calls over one document cost the service less - that tier is meant for testing, not for production documents. `ttl=0` stores nothing. Reading a resource address is asking to read, and needs no `ttl`. A request that carries a `password` is never cached: it neither reads a stored conversion nor stores one, so a document that was unlocked with a secret is never handed to a caller who did not name the same secret. On `POST /a2a` a task record is kept so that `GetTask` and `SubscribeToTask` can answer.

### Why route through the edge at all

Sending raw documents straight into a frontier model works, and for one small page it is often the right call. The trade changes with volume, size, and repetition:

- **Payload.** A webpage or a PDF carries layout, navigation, and encoding a model cannot use. Converting first means the model is paid for reasoning about content instead of for reading furniture.
- **Context.** A converted document is something a context window can actually hold; a raw one competes with the conversation it is part of.
- **Division of labour.** Extraction and transformation do not need a frontier model. Handing them to one is the most expensive available way to remove boilerplate.
- **Uniformity.** One conversion contract across webpages, PDFs, office documents, spreadsheets, archives, and legacy formats means one integration instead of one per format.
- **Addressing.** You can locate before you buy, and read one part of a large document without paying for the rest of it.

The promise is deliberately a narrow one, and it is the one this service can keep: remove weight before reasoning. It is not a substitute for a model, and no conversion makes a poor source good.

## What you get back

Depending on how you call the service, mdapi.io can return:

- Markdown - cleaned and normalized content, ready to read or feed into an LLM.
- Prompt results - summaries, extractions, or transformations based on your instructions.
- Both together - original conversion plus prompt-driven result when you need full control.
- Streaming output - chunked responses for large inputs or real-time UX.

The goal is always the same: give you just enough data to be useful, without wasting context or tokens.

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
