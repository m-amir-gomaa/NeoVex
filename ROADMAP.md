# NeoVex Roadmap

This document has two tracks that run in parallel: the **project roadmap** (what gets built and when) and the **personal learning track** (what I need to learn to build it). They are not separate — the learning track feeds directly into the build track.

The schedule constraint is real: 2–3 hours a day. Every phase is sized to respect that.

---

## Project Roadmap

### Phase 0 — Architecture and Vision (current)

**What:** Lock the system design before writing production code. Define every subsystem, its interfaces, and its responsibilities. Build the architecture diagrams. Run the idealization protocol with Antigravity.

**Completion signal:** `IDEALS.md` is written, reviewed, and canonized. Every subsystem has a design proposal. Branching strategy is agreed.

**Artifacts:** `IDEALS.md`, `/design/diagrams/`, `/design/step-*.md`

---

### Phase 1 — Minimal Wire (months 1–3)

**What:** The smallest possible end-to-end path. A Rust binary that accepts a prompt over HTTP, calls the Gemini API, and returns the response to a Neovim buffer. No RAG, no multi-agent, no MCP. Just the wire working correctly.

**Subsystems touched:** `rust_agent_server` (skeleton), `neovim-bridge` (skeleton), `model-router` (single provider, no fallback logic yet)

**Completion signal:** I can type a prompt in Neovim, hit a keybinding, and see a Gemini response appear in a buffer. The Rust server handles the request. The round-trip works reliably.

---

### Phase 2 — Agent Core (months 3–6)

**What:** Replace the single-call skeleton with a real agent loop. Implement the `rig` pipeline, `agentai` ToolCaller, and `StateManager`. The agent can now execute multi-step tasks with tool calls.

**Subsystems touched:** `agent-orchestrator`, `tool-caller`, `state-manager`

**Completion signal:** I can trigger a multi-step refactoring task from Neovim. The agent plans, calls tools (read file, write file), and applies changes. The loop runs without me intervening.

---

### Phase 3 — RAG Pipeline (months 5–8)

**What:** The agent can now retrieve context from the codebase before reasoning. Implement codebase indexing, vector storage, and cloud embedding API integration.

**Subsystems touched:** `rag-pipeline`, cloud embedding client, local embedding fallback

**Completion signal:** The agent answers questions about the codebase that it couldn't answer from the prompt alone. Retrieval is visibly improving response quality.

---

### Phase 4 — Model Router and Budget Control (months 7–10)

**What:** Multi-provider routing with real budget enforcement. The system tracks spending, respects caps, falls back gracefully, and surfaces routing state in the Neovim statusline.

**Subsystems touched:** `model-router` (full implementation), budget tracker, Ollama fallback, UI indicators

**Completion signal:** I can set a monthly budget cap, exhaust one provider's quota, and watch the system fall back with a visible warning — without the agent losing its state.

---

### Phase 5 — MCP Integration (months 9–12)

**What:** Connect the Rust server to external data sources via MCP. Implement the MCP client, configure bundled servers (filesystem, git, web search, LSP bridge), and enforce Sampling governance through the model router.

**Subsystems touched:** `mcp-layer`, bundled MCP server configs, Nix packaging for MCP servers

**Completion signal:** The agent can read from git history, fetch a web page for context, and query the LSP — all through MCP — without any of those calls bypassing the budget layer.

---

### Phase 6 — Mentorship System (months 11–14)

**What:** Implement the learning-state tracker and explanation-depth controller. The system builds a model of what I know and adapts its output accordingly.

**Subsystems touched:** `mentorship-system`, learning state persistence, concept translation layer

**Completion signal:** Explaining a new concept produces output calibrated to my background — it does not explain what a pointer is, it does explain what a monad is.

---

### Phase 7 — Hardening and Community (months 13–18)

**What:** Documentation, Nix packaging, plain-Linux installation path, contributor onboarding, and the first public release that someone other than me can install.

**Completion signal:** Someone who has never heard of NeoVex can install it from the README instructions and get the minimal wire working in under an hour.

---

## Personal Learning Track

This is not a course syllabus. It is the minimum knowledge I need to build each project phase, structured for 2–3 hours a day with a systems background (CSAPP, Linux internals). Things I already know are not listed.

---

### Track 1 — Rust Fluency (runs during Phase 0–1, ~2 hrs/day)

**Goal:** Write a working `axum` HTTP server with async handlers from memory.

**Resources, in order:**
1. [The Rust Book](https://doc.rust-lang.org/book/) — chapters 1–10 for ownership/borrowing. Skip anything about what a pointer is. Focus on chapters 4, 8, 9, 10.
2. [Rustlings](https://github.com/rust-lang/rustlings) — muscle memory for syntax. Do 2–3 exercises per session alongside the Book.
3. Jon Gjengset — [Crust of Rust](https://www.youtube.com/@jonhoo) on YouTube — lifetimes, iterators, trait objects. Watch after finishing the Book.

**Completion signal:** I can write a working `axum` server with a POST route, a JSON request body (using `serde`), and an async handler — from memory, without looking up the API.

---

### Track 2 — Async Rust (runs during Phase 1–2, ~1 hr/day)

**Goal:** Reason about `tokio` internals without looking them up.

**Resources:**
1. [Tokio tutorial](https://tokio.rs/tokio/tutorial) — official, covers tasks, spawning, channels, `select!`
2. Jon Gjengset — [Decrusting the tokio crate](https://www.youtube.com/watch?v=o2ob8zkeq2s) — how the runtime actually works
3. [async-book](https://rust-lang.github.io/async-book/) — for the underlying `Future` trait model

**Completion signal:** I can explain what `tokio::spawn`, `select!`, `Mutex`, and `mpsc` do at the level of the task scheduler — not just how to use them.

---

### Track 3 — Agentic AI Theory (runs during Phase 1–3, ~1 hr/day)

**Goal:** Understand what a ReAct loop does at the level of individual tokens and tool calls.

**Resources, in order:**
1. [ReAct paper](https://arxiv.org/abs/2210.03629) — Yao et al., 2022. The foundational loop NeoVex's ERS is based on. Read the full paper, not a summary.
2. [Toolformer paper](https://arxiv.org/abs/2302.04761) — Schick et al., 2023. How tool calls are integrated into generation.
3. [DeepLearning.AI — AI Agents in LangGraph](https://www.deeplearning.ai/short-courses/ai-agents-in-langgraph/) — short course, practical implementation of agent patterns. Free.
4. [AutoGen paper](https://arxiv.org/abs/2308.08155) — multi-agent coordination. Read after the above.

**Completion signal:** I can describe the difference between a ReAct agent and a plan-and-execute agent, explain why tool call latency matters architecturally, and design a simple agent loop on a whiteboard.

---

### Track 4 — Rust Crate Stack (runs during Phase 2–3, alongside building)

**Goal:** Be fluent in the specific crates NeoVex uses.

**Resources:**
- `rig` — [docs.rs/rig-core](https://docs.rs/rig-core) + source code on GitHub
- `axum` — [docs.rs/axum](https://docs.rs/axum) + official examples repo
- `mlua` — [docs.rs/mlua](https://docs.rs/mlua) + the `neovim-lib` examples
- `serde` — [serde.rs](https://serde.rs) — the derive macros are the important part

**Approach:** For each crate, read the docs once end-to-end, then build the smallest possible thing that uses it before using it in NeoVex.

---

## Where Contributors Can Slot In

If you want to contribute before Phase 1 production code exists, the highest-value work right now is in the design layer:

- **Agentic AI researchers** — review the ERS design in `/design/` and surface what the academic literature says about the open questions in [RESEARCH.md](./RESEARCH.md)
- **Rust engineers** — review the crate stack choices in the protocol and the class diagram; flag anything architecturally unsound before it gets built
- **Neovim plugin developers** — the `neovim-bridge` subsystem design needs someone who has written a serious Neovim plugin before
- **RAG specialists** — the RAG pipeline design in Phase 3 is the least locked-in subsystem; input on chunking strategy and embedding model selection is valuable now

Discussions happen on the [neovex-dev mailing list](https://groups.google.com/g/neovex-dev).
