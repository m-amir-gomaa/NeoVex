# NeoVex

> A cloud-native AI orchestration layer for Neovim, built in Rust, optimized for NixOS.

---

## Why This Exists

Most AI coding tools are built for the IDE majority — VS Code users, GUI workflows, managed environments. They assume you want a polished product that abstracts away how everything works underneath. NeoVex is built on the opposite assumption.

If you live in Neovim and run NixOS, you already made a deliberate choice: you want to understand your tools, compose them yourself, and own your environment completely. AI assistance shouldn't break that contract. It should fit into it.

NeoVex brings agentic AI capabilities — multi-step reasoning, tool execution, codebase retrieval, context-aware mentorship — directly into Neovim, without requiring you to open a different editor or surrender control of your workflow. The entire system is declared in your NixOS config, reproducible across machines, and always within your budget because you set the budget.

It's also built as a learning project. The author is actively studying systems programming and computer science while building this. NeoVex is designed to teach as it assists — not by hiding complexity, but by surfacing it at the right depth for someone who already understands memory, processes, and the Linux kernel, but is still mapping that knowledge onto software architecture and AI systems.

---

## Relation to Antigravity

[Antigravity](https://deepmind.google/antigravity) is Google's agent-first IDE — a fork of VS Code built around Gemini 3 Pro, with multi-agent orchestration, browser integration, workspace-aware reasoning, and deep task delegation baked into the editor itself. It represents the current ceiling of what an AI-native development environment looks like.

NeoVex is an explicit attempt to replicate most of Antigravity's capabilities — and where possible, exceed them — but entirely within the Neovim ecosystem, without the VS Code dependency, and with a fundamentally different architecture underneath.

The goal is not a port. It is a reimplementation from better foundations:

| | Antigravity | NeoVex |
|---|---|---|
| **Editor base** | Electron / VS Code | Neovim — no GUI framework overhead |
| **Source** | Closed, Google-controlled | GPL v3 — permanently open |
| **Runtime** | Node.js / Python | Rust — no GIL, true concurrency |
| **Agent logic** | Lives inside the IDE process | Standalone Rust server — editor-agnostic |
| **Model coupling** | Tied to Gemini | Provider-agnostic — Gemini, Claude, GPT-4, or any future model |
| **Configuration** | GUI settings, not reproducible | NixOS-native — entire system declared in your flake |
| **Budget control** | Opaque | Full user control over model hierarchy, costs, and fallback behaviour |

**Antigravity features NeoVex targets, in priority order:**

- **Multi-agent orchestration** — parallel sub-agents working on decomposed tasks simultaneously
- **Workspace-aware reasoning** — the agent understands your entire codebase, not just the open file
- **Task delegation** — break a high-level instruction into sub-tasks, execute them, reconcile results
- **Browser integration** — agents that can fetch documentation, search, and read external context
- **Inline code generation and editing** — suggestions applied directly to buffers with diff preview
- **Persistent agent memory** — context that survives across sessions, not just the current conversation
- **Learning and explanation mode** — adaptive depth, reasoning exposed rather than hidden

Where NeoVex goes further: the orchestration layer is a standalone Rust binary, fully decoupled from any editor. Neovim is the first-class target, but any editor that speaks HTTP can use it. Antigravity is architecturally inseparable from VS Code. NeoVex is not inseparable from anything.

---

## What It Does

- **Agentic reasoning in Neovim** — multi-step AI tasks (refactor, explain, debug, generate) that run as background agents, report progress live, and apply changes to your buffers
- **Cloud-first model routing** — always uses the best available cloud model (Gemini, Claude, GPT-4), falls back to local Ollama only when quotas are exhausted or a budget cap is hit, and tells you loudly when it does
- **RAG over your codebase** — indexes your project and retrieves relevant context before every inference call, using cloud embedding APIs with local fallback
- **MCP integration** — connects to external data sources (filesystem, git, web, LSP) via the Model Context Protocol, extending agent capabilities without coupling to any single provider
- **NixOS-native** — the entire system, including API keys, budget limits, model preferences, and background service configuration, is declared in your flake and managed by home-manager
- **Learning-oriented** — explanation depth adapts to your knowledge level; the system knows what you already know and builds on it rather than starting from scratch

---

## How It Will Evolve

NeoVex is a long-horizon project. It won't be finished in a month or a year. Here is the honest trajectory:

**Near term (months 1–6):** The agentic core gets built — the Rust server, the Neovim bridge, the model router, the basic RAG pipeline. This phase is about getting something that works end-to-end for a single task, not something feature-complete. The architecture decisions are being made now, carefully, so they don't need to be unmade later.

**Medium term (year 1–2):** The system deepens. MCP integration matures. The mentorship subsystem develops a real model of what the user knows and what they're learning. Multi-agent parallelism becomes practical. NixOS packaging becomes polished enough that someone else could install it from a flake with minimal friction.

**Longer term (years 2–5):** NeoVex becomes a real alternative for the class of developer who finds Neovim + NixOS to be the right environment. The plugin ecosystem grows. Other people contribute MCP servers, model router configurations, RAG strategies. The mentorship system accumulates enough translation mappings to be genuinely useful for CS learners coming from a systems background.

**Further out:** Unclear and intentionally so. The goal is not to ship a product — it is to build something that remains useful, composable, and honest about what it is for as long as someone finds value in it. If the AI landscape shifts completely in three years, NeoVex should be architected loosely enough to shift with it.

The one thing that won't change: this project will always be free and open-source. There is no business plan, no roadmap to a paid tier, no intention of ever extracting value from the people who use it. If it helps you, that is the whole point.

---

## Architecture

NeoVex is built around a local Rust orchestration server (`rust_agent_server`, running on `localhost:7777`) that handles all agent logic and calls cloud APIs for inference. Neovim communicates with it through a Lua/Rust plugin via `mlua`. The server never does local inference itself — that always goes to the cloud. Ollama is a last-resort fallback, never a preference.

Four canonical architecture diagrams live in [`/design/diagrams/`](./design/diagrams/) as PlantUML sources. Render them at [kroki.io](https://kroki.io) or with a local PlantUML installation:

| Diagram | What it shows |
|---|---|
| `component-diagram.puml` | High-level system components and connections |
| `sequence-diagram.puml` | Step-by-step flow of an agentic task |
| `class-diagram.puml` | Rust struct/trait hierarchy for the agentic core |
| `deployment-diagram.puml` | Runtime topology across local machine and cloud |

The full vision document (`IDEALS.md`) will be written and locked after the idealization protocol completes. The protocol itself lives in [`NeoVex-Antigravity-Prompt-v0_1_0.md`](./NeoVex-Antigravity-Prompt-v0_1_0.md).

---

## Status

Early architecture phase. The idealization protocol is running. No production code yet.

---

## License

GPL v3 — see [LICENSE](./LICENSE).

This means: use it, modify it, distribute it freely — but any derivative work must also be open-source under the same license. No one can take NeoVex, build on it, and close the source. The ecosystem stays open.