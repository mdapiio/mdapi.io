# About service

mdapi.io - Minimal Data API I/O: a content transformation layer primitive for AI systems.


Transforms documents, images, and webpages into AI-ready Markdown and structured data, optimized for LLM efficiency and token usage.

## What mdapi.io does

mdapi.io takes webpages, documents, images, and raw text and returns exactly what your model or agent needs:

- Clean Markdown instead of raw HTML or PDF clutter.
- Structured data when you need fields, tables, or entities.
- Prompt-driven summaries, extractions, or analyses with custom instructions.
- Streaming output for large inputs or real-time workflows.

All supported protocols and interfaces ultimately converge on the same transformation core.

## Edge-native design

mdapi.io is built for stateless edge execution.

- Processing happens in memory.
- No user data is stored after conversion.
- The service reduces context, token, and infrastructure usage by moving heavy extraction and transformation work to the edge.

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

Each request is handled by a stateless, automatically-scaled execution environment, so the service scales horizontally - the more agents that call it, the more it parallelizes. An orchestrator can fan work out across a swarm of agents, and the swarm processes very large batches of distinct resources in parallel. Because every protocol converges on the same core, agents can cooperate directly and hand compact intermediate results to one another.

In practice this means millions of resources can be processed in a matter of minutes, not hours - the practical ceiling is set by how widely the orchestrator distributes the work, not by the service itself. Different users may freely access the same resource; each user simply stays within their own fair-use allowance.

### Shared vs individual payment

- **Shared token** - the orchestrator pays once, activates a token for the needed volume, and hands it to the whole swarm. This batches on-chain activity and reduces blockchain load.
- **Individual tokens** - each agent pays for its own work, activating exactly the volume it received (e.g. 166 resources → pay for 166 upfront).
- If a wallet is missing or underfunded, the agent drops into **human-in-the-loop** mode: it returns payment details and a QR code, the human pays from a mobile device, and the agent resumes automatically on confirmation.

### Role switching and multi-agent / human collaboration

An agent's role can change mid-task. One agent may fetch and normalize a webpage, then hand the compact Markdown to another agent (via the text or prompt parameters) that summarizes or extracts. The same flow works between agent and human: an IDE agent indexes scattered project documentation in mixed formats and surfaces sourced snippets; a research agent prepares RAG-ready input; a coordinator delegates and reassembles results.

### Cross-protocol interoperability

Because REST, MCP, ACP, A2A, and OpenAI-compatible endpoints all call the same core, different agent types can cooperate directly. Knowledge extracted on one protocol can be relayed on another - the transport is interchangeable, the result is the same.

### Bulk processing and model training

For any task that needs maximum resources processed in minimum time (LLM training corpora, large-scale RAG indexing, bulk document normalization), the swarm pattern turns mdapi.io into a high-throughput edge pipeline.

### Edge filter in front of expensive models

The core job of mdapi.io is to remove weight before it reaches reasoning. Instead of sending a raw PDF, webpage, or scanned document straight into an LLM, an agent routes it through mdapi.io first and forwards only the compact result. That keeps context windows small, cuts token spend, and avoids wasting compute on navigation, boilerplate, and layout the model never needed. The service is the heavy-lifting layer; the model gets just the part it must reason about.

### Prompt as a command between agents

The `prompt` parameter is not only instructions for the service - it is also how one agent directs another. An agent can hand a processed resource to a peer together with a prompt like "extract only the price tables" or "return this as JSON", and the receiving agent lets mdapi.io execute that instruction. The service becomes the shared execution engine for the swarm's intent, not just a converter.

### Legacy to AI-ready bridge

Many real systems still speak in scans, spreadsheets, and old document formats. mdapi.io normalizes that mixed, legacy input into clean AI-ready Markdown or structured data on the fly, so agents can treat a decades-old report the same way they treat a modern webpage. It is the translation layer that lets older infrastructure participate in an agentic workflow without refactoring.

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


> mdapi.io is an edge-native service-transport primitive for AI, autonomous-agents, and the Web4 ecosystem.
