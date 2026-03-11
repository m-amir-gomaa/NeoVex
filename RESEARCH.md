# NeoVex — Open Research Questions

Specific technical questions NeoVex needs answered before or during
implementation. Not general calls for help — concrete problems where the
right answer changes an architectural decision.

To engage: open an issue referencing the question number, or reach out
via the mailing list.

> **Note:** Implementation has not started yet. These questions are being
> collected now so that researchers can engage during the design phase,
> before decisions get locked into code.

---

## R-001 — Mutex vs RwLock for Agent State Under Async Workloads

**Context:** The `StateManager` uses `Arc<Mutex<HashMap<String, Value>>>`
for shared state across concurrent sub-agents in tokio. This serializes
all state access.

**Question:** For a multi-agent system with bursty reads and rare writes,
is `Arc<RwLock<T>>` strictly better? Are there workloads where the
read/write contention pattern of a ReAct reasoning loop causes `RwLock`
to be slower due to writer starvation or fairness behaviour on Linux —
specifically under `tokio::sync`, not `std::sync`?

**Why it matters:** `StateManager` design decision. Wrong choice means
unnecessary serialization or starvation bugs under concurrent agent load.

**What I need:** Benchmarks or analysis of `Mutex` vs `RwLock` under
bursty-read, rare-write workloads in async Rust using `tokio::sync`.

---

## R-002 — ReAct Loop Latency vs. Streaming in a Neovim Context

**Context:** Each ERS reasoning step requires a cloud API round-trip.
At 1–4 seconds per call, a 5-step loop is 5–20 seconds before anything
appears in the buffer.

**Question:** Does streaming reasoning tokens to the user as they arrive
meaningfully reduce perceived abandonment even when total wall time is
identical? What does the HCI literature say about incremental vs. batch
delivery for AI coding tools?

**Why it matters:** Streaming mid-loop requires a fundamentally different
Neovim buffer update model than batch delivery. Must be decided before
Phase 2.

**What I need:** HCI research on perceived latency in agentic tools, or
prior art from tools that have shipped both modes and measured behaviour.

---

## R-003 — MCP vs. Native Tool Use for the ERS Tool Layer

**Context:** Cloud LLMs have native tool use / function calling APIs.
MCP also describes tools. NeoVex currently designates MCP as the
external data layer only, with native LLM tool use handling ERS tool
calls directly. This creates two tool registries.

**Question:** Is there a principled reason to route all ERS tool calls
through MCP, treating native LLM tool use as an implementation detail
underneath? Has anyone built a system that unifies these two layers
cleanly?

**Why it matters:** If the answer is "unify through MCP," the `ToolCaller`
implementation changes significantly. Must be decided before Phase 2.

**What I need:** Prior art or architectural analysis of systems that tried
both approaches — particularly latency of MCP overhead vs. native tool use.

---

## R-004 — AST-Aware Chunking for RAG in Mixed Rust/Lua Codebases

**Context:** NeoVex's RAG pipeline chunks code for indexing. Naive
line/token chunking splits functions arbitrarily. AST-aware chunking
respects structure but requires a parser per language. Tree-sitter is
already in Neovim.

**Question:** What is the right chunking strategy for a codebase mixing
Rust and Lua? Is tree-sitter the right foundation, and what does a good
Rust implementation of tree-sitter-based chunking look like at the
function level?

**Why it matters:** Chunking strategy is one of the highest-leverage
decisions in RAG quality and one of the hardest to change after the
index is built.

**What I need:** Benchmarks comparing chunking strategies on code
retrieval tasks, or existing Rust implementations of AST-aware chunking
that could be adapted.

---

## R-005 — Persistent Agent Memory Across Sessions Without a Hosted Service

**Context:** NeoVex targets persistent memory locally — context surviving
across sessions. Naive history dump hits context window limits. Summarisation
loses detail. Vector retrieval of past sessions adds latency at start.

**Question:** What is the current state of the art for long-term agent
memory in a fully local context? Is there a Rust-compatible library for
episodic memory management, or does this need building from scratch?

**Why it matters:** Both `StateManager` and the `mentorship-system`
depend on this. The mentorship system needs to remember explained concepts
and at what depth, indefinitely.

**What I need:** Survey of persistent memory approaches for local AI
agents, or prior art from projects that have shipped this at any scale.

---

## R-006 — Budget Enforcement at the MCP Sampling Layer

**Context:** MCP's Sampling primitive lets MCP servers request LLM
completions through the host client, potentially bypassing NeoVex's
`ModelRouter` budget tracking.

**Question:** How do existing MCP host implementations (Cursor, Claude
Desktop, etc.) handle Sampling governance? Is there prior art for a host
that enforces budget caps on Sampling calls?

**Why it matters:** Architectural vulnerability — a misconfigured MCP
server can exhaust cloud quota silently. Must be fixed in the MCP client
before any servers are enabled.

**What I need:** Review of existing MCP host implementations and how they
handle — or fail to handle — Sampling budget governance.
