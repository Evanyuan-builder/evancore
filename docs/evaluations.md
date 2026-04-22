# Ecosystem Evaluations

> Every external project considered for absorption into EvanCore — and every one that was declined — is logged here. This log exists so the project's scope stays defensible.

---

## The framework (5 dimensions)

When a candidate repo is proposed, it's evaluated against:

1. **Fit** — Which EvanCore line does this land on?
   - `assembly` (HarnessSpec / Assembler / validator)
   - `delivery` (SpecStore / release / overlay / promote / upgrade / registry / signing / trust)
   - `observability` (EventBus / exporter / OTel)
   - `open foundation` (Role / Skill / Memory / Tool backend)
   - **`runtime` — off-scope by default** (v1 strategic lock, 2026-04-21)

2. **Gap reality** — Is this a known gap, or a new scope?
   - Known gaps: MCP hot-reload, B3 Phase C (CRL / cross-host revocation / registry self-signing), second Memory backend, B3 Phase A.2 (`_index.json` per-spec current / retry jitter)
   - Anything else = new scope, requires explicit acknowledgment

3. **Absorption form** — pick one:
   - **(a) idea reference only** (cheapest, most common)
   - **(b) vendor subset into EvanCore**
   - **(c) optional dependency** (pyproject.toml extras)
   - **(d) decline** (off-fit / too costly / conflicts with invariants)

4. **Cost side** — license compat / activity / API stability / dependency footprint

5. **Invariant conflict check** — per-backend instance (no process-level singletons), harness/team atomic release unit, `assemble ≠ release`, Tool reverse-lookup vs Role/Skill/Memory `scheme://` forward-dispatch split, `_async_*` monkey-patch hook points, silent degradation semantics (MCP unreachable → 0 tools + warn, not crash)

**Default bias: (a) or (d).** (b) and (c) require a specific justification.

---

## Evaluated candidates (2026-04-22 batch)

Eleven repos evaluated in one sitting, during the robust-phase kickoff.

### Accepted as reference ideas

| Repo | Line | Verdict | Reason |
|---|---|---|---|
| [`vectorize-io/hindsight`](https://github.com/vectorize-io/hindsight) | Memory (open foundation) | **(a) idea reference**, (c) candidate if/when second Memory gap activated | MIT, Python, v0.5.3. retain/recall/reflect three-op model + biomimetic structure + LongMemEval benchmark. **Integration shortcut**: Hindsight ships its own MCP server — can be consumed via EvanCore's existing `mcp://` Tool backend with zero EvanCore changes (lands on Tool line, not Memory line). Won't push absorption until second-Memory gap is user-activated. |
| [LLMWiki v2 (gist)](https://gist.github.com/rohitg00/2067ab416f7bbe447c1977edaaa681e2) | Memory (open foundation) — concept only | **(a) idea reference** | Conceptual blueprint (not code): memory consolidation tiers (working → episodic → semantic → procedural), confidence scoring, supersession tracking, retention decay. Useful primitives for a future second Memory backend. Copying verbatim would balloon scope into PKM territory — not doing that. |

### Declined

Each of these declines for one of three reasons: *off-fit* (lands on `runtime` or outside any EvanCore line), *wrong abstraction layer* (solves a problem EvanCore doesn't own), or *terminology collision* (uses "spec" or "harness" to mean something incompatible).

| Repo | Category | Decline reason |
|---|---|---|
| [`EvoAgentX/EvoAgentX`](https://github.com/EvoAgentX/EvoAgentX) | Agent framework | Runtime concern — off-fit with v1 scope lock. |
| [`multica-ai/multica`](https://github.com/multica-ai/multica) | Multi-agent orchestration | Runtime + orchestration — both off-fit. |
| [`VectifyAI/PageIndex`](https://github.com/VectifyAI/PageIndex) | Document indexing | Solves a different problem (retrieval over docs), doesn't intersect the lifecycle. |
| [`safishamsi/graphify`](https://github.com/safishamsi/graphify) | Graph-based agent flow | Orchestration DAG primitive — wrong layer. |
| [`alibaba/page-agent`](https://github.com/alibaba/page-agent) | Web-automation agent runtime | Pure runtime. |
| [`thedotmack/claude-mem`](https://github.com/thedotmack/claude-mem) | Claude Code memory wrapper | Tool/UX wrapper, not a reusable backend. If users want it, they plug it in via their own environment — EvanCore doesn't need to know. |
| [`shanraisshan/claude-code-best-practice`](https://github.com/shanraisshan/claude-code-best-practice) | Practice collection | Documentation, not a component. No absorption shape. |
| [`gsd-build/get-shit-done`](https://github.com/gsd-build/get-shit-done) | Spec-driven coding workflow | **Terminology collision warning**: uses "spec" and "harness" to mean prompts for code generation, not agent configuration. Same words, different products. External messaging needs to pre-empt this confusion. |
| [`snarktank/ralph`](https://github.com/snarktank/ralph) | Agent orchestration loop | Runtime / orchestration — off-fit. |

---

## Why the default is decline

EvanCore's v1 scope lock (2026-04-21) excludes `runtime` concerns. The robust phase explicitly does **not** proactively expand into runtime capabilities or pre-conditional gap items (MCP hot-reload, distribution Phase C).

Most repos that look "AI-adjacent" land on runtime or orchestration, which are already well-served by frameworks whose primary job is exactly that. Absorbing them would dilute EvanCore's one defensible position: the delivery layer between spec and running agent.

**Declining a promising repo is not a rejection of the repo — it's a refusal to let EvanCore drift.**

---

## What would change a verdict

A future candidate gets a stronger look if it:

- Lands on an **activated gap** (second Memory backend, B3 Phase A.2, distribution Phase C, MCP hot-reload)
- Introduces a **new lifecycle primitive** that EvanCore's current assemble/release/promote/rollback/upgrade shape doesn't cover — and the primitive is general (not vendor-locked)
- Offers a **signature / trust / revocation** improvement that's strictly additive to the current Ed25519 + per-host trust + revocation v2 model

Anything else stays at (a) or (d).
