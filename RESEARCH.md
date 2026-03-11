# NeoVex — Open Research Questions

This document lists the specific technical questions NeoVex needs answered before or during implementation. These are not vague calls for help — they are concrete problems where the right answer would change an architectural decision.

If you have depth in any of these areas and want to engage, open an issue referencing the question number, or join the discussion on the [neovex-dev mailing list](https://groups.google.com/g/neovex-dev).

---

## R-001 — Agent State Consistency Under Concurrent Tool Calls

**Context:** The `StateManager` in NeoVex uses `Arc<Mutex<HashMap<String, Value>>>` to manage shared agent state across concurrent sub-agents. This is the standard Rust approach, but it serializes all state access.

**The question:** For a multi-agent system where sub-agents frequently read state but rarely write it, is `Arc<RwLock<T>>` strictly better than `Arc<Mutex<T>>`? Are there workloads where the read/write contention pattern of an agentic reasoning loop would make `RwLock` actually slower due to writer starvation or fairness issues on Linux?

**Why it matters:** This decision is in the `StateManager` design. Getting it wrong means either unnecessary serialization (too conservative) or starvation bugs under load (too aggressive).

**What we need:** Benchmarks or analysis of `Mutex` vs `RwLock` under bursty-read, rare-write workloads in async Rust. Ideally referencing `tokio::sync` specifically, not `std::sync`.

---

## R-002 — ReAct Loop Latency vs. Streaming in a Neovim Context

**Context:** NeoVex's ERS uses a ReAct-style loop — the agent generates a thought, calls a tool, observes the result, and repeats. Each step requires a cloud API round-trip. With Gemini or Claude, a single non-streaming call takes 1–4 seconds. A 5-step ReAct loop is therefore 5–20 seconds of wall time before the user sees anything.

**The question:** Is the right answer to stream the reasoning tokens to the user as they arrive (showing the agent "thinking" in real time), or to run the full loop silently and only surface the final result? What does the research say about user perception of latency in agentic tools — does seeing incremental progress reduce abandonment even when total wall time is identical?

**Why it matters:** This is a UX decision that also has implementation consequences. Streaming mid-loop requires a different Neovim buffer update model than batch delivery.

**What we need:** Any HCI research on perceived latency in AI coding tools, or prior art from tools that have shipped both modes and measured user behaviour.

---

## R-003 — MCP vs. Native Tool Use for the ERS Tool Layer

**Context:** Cloud LLMs (Gemini, Claude, GPT-4) have native tool use / function calling APIs. MCP is a separate protocol that also describes tools. NeoVex currently designates MCP as the external data layer only, with native tool use handling ERS tool calls directly. But this creates two tool registries.

**The question:** Is there a principled reason to route all tool calls through MCP (treating native LLM tool use as an implementation detail underneath), or does that add latency and complexity without benefit? Has anyone built a system that unifies these two layers cleanly?

**Why it matters:** If the answer is "unify them through MCP," the `ToolCaller` implementation changes significantly. This decision should be made before Phase 2.

**What we need:** Prior art, benchmarks, or architectural analysis of systems that have tried both approaches. Particularly interested in latency characteristics of MCP tool call overhead vs. native tool use.

---

## R-004 — AST-Aware Chunking for RAG in Multi-Language Codebases

**Context:** NeoVex's RAG pipeline needs to chunk codebases for indexing. Naive chunking by line count or token count splits functions and classes arbitrarily. AST-aware chunking respects code structure, but it requires a working parser for each language.

**The question:** What is the right chunking strategy for a codebase that mixes Rust, Lua, and potentially other languages? Is tree-sitter the right foundation (given it's already in Neovim), and if so, what does a good Rust implementation of tree-sitter-based chunking look like? What chunk granularity (file, function, block) produces the best retrieval precision for code Q&A tasks?

**Why it matters:** The chunking strategy is one of the highest-leverage decisions in RAG quality. It's also one of the hardest to change after the index is built.

**What we need:** Benchmarks comparing chunking strategies on code retrieval tasks, or a pointer to existing Rust implementations of AST-aware chunking that could be adapted.

---

## R-005 — Persistent Agent Memory Across Sessions

**Context:** NeoVex targets persistent agent memory — context that survives across sessions. The naive approach (dump conversation history to disk and reload it) hits context window limits quickly and degrades quality. Summarization loses detail. Vector retrieval of past sessions adds latency.

**The question:** What is the current state of the art for long-term agent memory in a local-first context (no hosted memory service)? Is there a Rust-compatible library for episodic memory management, or does this need to be built from scratch? What are the failure modes of each approach at 6-month timescales?

**Why it matters:** The `StateManager` and `mentorship-system` both depend on this. The mentorship system needs to remember what concepts have been explained and at what depth — and that memory needs to survive indefinitely, not just for a session.

**What we need:** Survey of persistent memory approaches for local AI agents, or prior art from projects that have shipped this at any scale.

---

## R-006 — Budget Enforcement at the MCP Sampling Layer

**Context:** MCP's Sampling primitive allows MCP servers to request LLM completions through the host client. This means a tool server can trigger cloud API calls that bypass NeoVex's `ModelRouter` budget tracking — unless the Sampling requests are explicitly intercepted and routed through the same budget enforcement layer.

**The question:** How do other MCP host implementations handle Sampling governance? Is there prior art for a host that enforces budget caps on Sampling calls, or does every implementation currently just pass them through untracked?

**Why it matters:** This is an architectural vulnerability. If MCP Sampling calls bypass the budget layer, a user can unknowingly exhaust their cloud quota through a misconfigured or malicious MCP server. The fix needs to be in the MCP client implementation before any MCP servers are enabled.

**What we need:** Review of existing MCP host implementations (Cursor, Claude Desktop, etc.) and how they handle or fail to handle Sampling budget governance.
