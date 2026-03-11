# NeoVex Roadmap

---

## Current Status: Learning Phase — Implementation Stalled

NeoVex is intentionally not being built yet. The architecture is designed.
The vision is being locked. Implementation does not start until the learning
roadmap below is complete.

This is a deliberate choice. I don't believe in fighting a compiler before
understanding what it's enforcing — that ingrains wrong patterns and wastes
time better spent learning correctly. NeoVex is the purpose, not the
classroom. The learning track below builds the foundation. Implementation
starts when that foundation is solid.

---

## Who I Am (Honest Starting Point)

- Finished CSAPP in 8 weeks — theory, exercises, the full book
- Strong theoretical CS and systems understanding
- JavaScript in practice ([Keep](https://github.com/m-amir-gomaa/Keep))
- C theory from CSAPP — deep understanding, no lab implementation
- Neovim environment fully configured: LSP, linter, formatter, debugger,
  keybindings — ready to write Rust the moment I understand it
- **Rust: day zero. Never written a line.**
- **Agentic AI: day zero. No prior study.**

Fast concept acquisition is real but it doesn't eliminate time. The
bottleneck here isn't intelligence — it's the accumulated judgment that
only comes from actually building things, and the consistency required to
sustain a long-horizon project on 2–3 hours a day. The timeline below
respects that honestly.

---

## Learning Roadmap (Pre-Implementation)

This runs before any NeoVex implementation code is written. Two tracks
in sequence, not in parallel.

---

### Track 1 — Rust Fundamentals (Weeks 1–4)

**Goal:** Solid intermediate Rust before touching NeoVex code.

Ownership and borrowing will map directly from CSAPP's memory model —
the concepts are not new, the compiler enforcing them at compile time is.
JavaScript experience means async/await syntax will feel familiar; the
difference is that Rust futures are lazy and the borrow checker applies.

**Week 1–2 — Core Language**

- [The Rust Book](https://doc.rust-lang.org/book/) — ch. 1–10, 13, 16–20
  Read in order. Do not skip the ownership chapters to get to "the
  interesting parts" — those chapters *are* the interesting parts for
  someone with a systems background.
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/) —
  run alongside the Book as a quick interactive reference

**Week 3–4 — Async and NeoVex-Specific Crates**

- [Async Book](https://rust-lang.github.io/async-book/) — Tokio async
  runtime, futures model
- [tower-lsp boilerplate](https://github.com/ebkalderon/tower-lsp-boilerplate)
  — read and understand a Neovim LSP server in Rust before writing one
- [Ratatui](https://ratatui.rs/) — TUI framework for the Neovim interface
  layer; read the examples

**Completion signal:** I can write an async Tokio runtime with multiple
tasks, understand what the borrow checker is rejecting and why, and read
the tower-lsp boilerplate and explain every line. Not write from scratch —
read and explain.

---

### Track 2 — Agentic AI Foundations (Weeks 5–8)

**Goal:** Enough understanding to implement the ERS designs the AI
specialist produces — not to design them myself.

I need to read LangGraph-style diagrams, ask good implementation
questions, implement bridges (model switching, tool wrappers, Ollama
client), and sanity-check what's designed against what's buildable.
Deep agent theory is not the target. Light but solid exposure is.

**Structured Courses (in order):**

1. **Google's 5-Day AI Agents Intensive** (Kaggle, free, self-paced)
   Foundations, architecture, evaluation, deployment.
   → [kaggle.com/learn-guide/5-day-agents](https://www.kaggle.com/learn-guide/5-day-agents)

2. **IBM RAG and Agentic AI** (Coursera, free to audit)
   Hands-on RAG pipelines, multimodal agents, vector stores.
   → [coursera.org — IBM RAG and Agentic AI](https://www.coursera.org/professional-certificates/ibm-rag-and-agentic-ai)

3. **Stanford CS329A: Self-Improving AI Agents** (materials free)
   Verifiers, tool use, memory. Google and Anthropic research.
   → [cs329a.stanford.edu](https://cs329a.stanford.edu/)

**Supplementary (alongside courses):**

- [OpenAI: Practical Guide to Building Agents (PDF)](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf)
- [Anthropic: Tips for Building AI Agents](https://www.youtube.com/watch?v=LP5OCa20Zpg)
- [NVIDIA: Building RAG Agents with LLMs](https://learn.nvidia.com/courses/course-detail?course_id=course-v1%3ADLI+C-FX-15+V1)

**Key papers (after courses):**

- [ReAct — Yao et al., 2022](https://arxiv.org/abs/2210.03629)
  The reasoning loop NeoVex's ERS is based on.
- [Toolformer — Schick et al., 2023](https://arxiv.org/abs/2302.04761)
  How tool calls integrate with generation.
- [AutoGen — Wu et al., 2023](https://arxiv.org/abs/2308.08155)
  Multi-agent coordination.

**Completion signal:** I can read an ERS design diagram, ask the right
implementation questions, and describe the difference between a ReAct
agent and a plan-and-execute agent without looking it up.

---

## Implementation Roadmap (Starts After Learning Track)

These phases do not start until both learning tracks are complete.
The Antigravity idealization protocol executes, contributors are invited,
and implementation begins together.

---

### Phase 0 — Vision Lock (concurrent with learning, no code)
**What:** Finalize `IDEALS.md` via the Antigravity protocol. Lock every
subsystem's design and interfaces. This runs while I study — it's
documentation and design, not implementation.

**Completion signal:** `IDEALS.md` canonized. All subsystem proposals
written and approved.

---

### Phase 1 — Minimal Wire
**What:** A Rust binary using `axum` that accepts a prompt over HTTP,
calls the Gemini API, and writes the response to a Neovim buffer via
`mlua`. No RAG, no agents, no MCP. The wire only.

**Completion signal:** I type a prompt in Neovim, hit a keybinding, see
a Gemini response in a buffer, and can explain every line of Rust that
makes it happen.

---

### Phase 2 — Agent Core
**What:** Replace the single call with a real agent loop. `rig` pipeline,
`agentai` ToolCaller, `StateManager` with concurrent state.

**Completion signal:** A multi-step refactoring task runs from Neovim
without me intervening. I understand the concurrent state model well
enough to reason about race conditions.

---

### Phase 3 — RAG Pipeline
**What:** Codebase indexing, vector storage, cloud embedding API.

**Completion signal:** The agent answers questions about the codebase it
couldn't answer from the prompt alone.

---

### Phase 4 — Model Router and Budget Control
**What:** Multi-provider routing, budget enforcement, Ollama fallback,
routing state in the Neovim statusline.

**Completion signal:** I exhaust one provider's quota and the system
falls back visibly without losing agent state.

---

### Phase 5 — MCP Integration
**What:** MCP client, bundled external data servers, Sampling governance.

**Completion signal:** The agent reads git history, fetches a web page,
queries the LSP — none bypassing the budget layer.

---

### Phase 6 — Mentorship System
**What:** Learning-state tracker, explanation-depth controller.

**Completion signal:** Explaining a concept produces output calibrated to
my background — skips what I know, explains what I don't.

---

### Phase 7 — Hardening and Community
**What:** Documentation, Nix packaging, plain-Linux install, first public
release.

**Completion signal:** Someone unfamiliar with NeoVex can install it from
the README and get the minimal wire working in under an hour.

---

## When Contributors Get Invited

Not now. When the learning roadmap is complete and Phase 0 (vision lock)
is done, I will begin actively seeking contributors. The design will be
solid enough by then for outside contributions to slot in without
destabilizing it.

The areas where contributors will matter most:

- **Agentic AI researchers** — address the open questions in
  [RESEARCH.md](./RESEARCH.md) and review ERS design
- **Rust engineers** — review crate stack and flag architectural issues
  before they get built
- **Neovim plugin developers** — the `neovim-bridge` subsystem needs
  someone who has shipped a serious Neovim plugin
- **RAG specialists** — chunking strategy and embedding model selection
  before Phase 3 locks
