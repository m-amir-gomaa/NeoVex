# NeoVex

> A cloud-native AI orchestration layer for Neovim, built in Rust, runs on any Linux distribution.

---

## Why This Exists

Most AI coding tools are built for the IDE majority — VS Code users, GUI workflows, managed environments. They assume you want a polished product that abstracts away how everything works underneath. NeoVex is built on the opposite assumption.

I live in Neovim and I care about owning my environment. AI assistance shouldn't break that contract — it should fit into it. NeoVex is my attempt to bring the full capability of agentic AI directly into Neovim, on any Linux distribution, without surrendering control of how it works underneath.

I'm building this while actively learning. I'm currently studying Rust and taking courses on agentic AI systems — the architecture of NeoVex is being designed in parallel with my own understanding of it deepening. Antigravity is my primary development environment for building NeoVex itself; the agent-assisted workflow I'm trying to replicate for Neovim users is the same one I'm using to build it. That's intentional — it means every capability I implement is one I've personally depended on and understand from the inside.

I review every line of code before it gets pushed. This is not a project where commits get merged without being read. I'd rather ship slowly and understand everything in the codebase than move fast and accumulate code I can't reason about. That standard will not change as the project grows.

NeoVex is also designed to teach as it assists — not by hiding complexity, but by surfacing it at the right depth. I come from a systems background (CSAPP, Linux internals) and I'm still mapping that knowledge onto software architecture and AI systems. The mentorship layer reflects that journey: it builds on what you already know rather than starting from scratch.

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
| **Configuration** | GUI settings, not reproducible | Nix-first — declarative config via flake, but usable on any Linux distro |
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
- **Nix-first, distro-agnostic** — the preferred installation path uses the Nix package manager for reproducible, declarative configuration of API keys, budget limits, model preferences, and background services; but NeoVex runs on any Linux distribution without Nix if you prefer to manage it yourself
- **Learning-oriented** — explanation depth adapts to your knowledge level; the system knows what you already know and builds on it rather than starting from scratch

---

## How It Will Evolve

NeoVex is a long-horizon project. It won't be finished in a month or a year. Here is the honest trajectory:

**Near term (months 1–6):** The agentic core gets built — the Rust server, the Neovim bridge, the model router, the basic RAG pipeline. This phase is about getting something that works end-to-end for a single task, not something feature-complete. I'm making the architecture decisions now, carefully, so they don't need to be unmade later.

**Medium term (year 1–2):** The system deepens. MCP integration matures. The mentorship subsystem develops a real model of what I know and what I'm learning. Multi-agent parallelism becomes practical. Nix packaging becomes polished enough that someone else could install it from a flake with minimal friction, while plain-Linux installation paths get documented and tested.

**Longer term (years 2–5):** NeoVex becomes a real alternative for any developer who finds Neovim to be the right environment, regardless of which Linux distribution they run. The plugin ecosystem grows. Contributors bring MCP servers, model router configurations, RAG strategies. The mentorship system accumulates enough translation mappings to be genuinely useful for learners coming from a systems background.

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

## Community

**Mailing list:** [neovex-dev@googlegroups.com](https://groups.google.com/g/neovex-dev)
This is the primary place for design discussions, research questions, contributor introductions, and announcements. Public archive, anyone can post by email without an account.

**Where the relevant communities are:**
- Agentic AI / LLM engineering — [r/LocalLLaMA](https://reddit.com/r/LocalLLaMA), [Hugging Face Discord](https://discord.gg/hugging-face)
- Rust systems programming — [r/rust](https://reddit.com/r/rust), [Rust Discord](https://discord.gg/rust-lang)
- Neovim — [r/neovim](https://reddit.com/r/neovim)

---

## Contributing

Not actively recruiting contributors yet — see [Status](#status).

When the learning roadmap is complete and the vision is locked, I will begin seeking contributors with depth in: agentic AI systems research, Rust systems programming, Neovim plugin development, and RAG pipeline design. The specific open questions researchers can engage with now are in [RESEARCH.md](./RESEARCH.md).

A few things to know before contributing when that time comes: I read every line of code before it merges, and I'll push back on anything I don't understand or don't agree with. That's not a gatekeeping stance — it's how I stay in command of what NeoVex is. If you submit a PR, expect a real review, not a rubber stamp.

If you want to follow along or start a conversation before then, the [mailing list](https://groups.google.com/g/neovex-dev) is the right place.

---

## Status

**Intentionally stalled — learning phase in progress.**

No implementation code will be written until the learning roadmap in
[ROADMAP.md](./ROADMAP.md) is complete. That means Rust fundamentals
(~4 weeks) followed by agentic AI foundations (~4 weeks). The
architecture design and vision lock via the Antigravity protocol runs
concurrently since it is documentation, not code.

Contributors and researchers are not being actively recruited yet.
When the learning roadmap is complete and `IDEALS.md` is canonized,
that changes. See [ROADMAP.md](./ROADMAP.md) for when and how.

Open questions that researchers can engage with now are in
[RESEARCH.md](./RESEARCH.md).

---

## License

GPL v3 — see [LICENSE](./LICENSE).

This means: use it, modify it, distribute it freely — but any derivative work must also be open-source under the same license. No one can take NeoVex, build on it, and close the source. The ecosystem stays open.