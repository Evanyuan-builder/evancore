# Philosophy

> EvanCore is not a runtime, not an orchestrator, and not an MCP platform.
> It is the delivery layer for agent harnesses.

This document exists because the name "agent *something*" triggers reflexive categorization, and EvanCore does not fit the categories it looks like it should fit.

---

## 1. The category doesn't exist yet, but it's obvious in retrospect

There is a general-purpose answer to: *how do I assemble this set of infra components into a running thing, version it, sign it, deploy it, reverse it, distribute it?*

For infrastructure, the answer is Terraform / Pulumi / OpenTofu.
For container images, the answer is Docker + OCI registries.
For database migrations, the answer is Alembic / Flyway / golang-migrate.
For serverless app bundles, the answer is AWS SAM / Cloudflare Wrangler.

For **agent harnesses** — the assembly of prompts, tools, memory, guardrails, policies, and identity into something deployable — there is no standard answer yet. The current answers are:

- **Agent frameworks** (LangGraph, CrewAI, Microsoft Agent Framework) — these are *runtimes*. They solve "how does the agent execute a task," not "how does this specific configured harness get shipped."
- **Hosting platforms** (Modal, Fly, Vercel AI) — these are *runtimes-as-a-service*. They run what you already built.
- **MCP** — this is an *integration substrate*. It connects an agent to external tools. It doesn't say anything about what a harness *is*, how it's versioned, or how it's signed.

EvanCore is the missing layer: the delivery layer between the spec file and the running agent.

---

## 2. The lifecycle is the differentiator

EvanCore does seven things that, together, form a lifecycle:

```
spec ──▶ validate ──▶ assemble ──▶ release ──▶ [ promote | rollback | upgrade ] ──▶ publish ──▶ pull ──▶ verify
```

Each operation has specific guarantees:

| Op | Guarantee |
|---|---|
| `validate` | Schema conformance; reference closure; no dangling tool/skill refs |
| `assemble` | Deterministic artifact from spec — same spec, same hash |
| `release` | Content-addressed storage (blake2b-64 prefix) + optional Ed25519 signature |
| `promote` | Deploy artifact from one env to another; source env is the invariant, not the spec |
| `rollback` | Re-deploy the *previous* content-addressed artifact; no spec rewrite |
| `upgrade` | Move a running harness to a new artifact version with a defined upgrade target |
| `publish` / `pull` | Push to / pull from a registry (file:// or https://) with federation |
| `verify` | Re-check signature against per-host trust.yaml + revocation v2 list |

**None of these are runtime operations.** The agent doesn't care. The developer does. This is the distinction between "my agent works" and "my agent is *operationally sound*."

---

## 3. CLI is the surface for a reason

Developers meet the agent stack where they already meet everything else: the terminal.

- Anthropic ships Claude Code as "a coding agent that lives in your terminal."
- Microsoft pairs Agent Framework 1.0 with Azure Developer CLI.
- Hugging Face ships `huggingface-cli`. OpenAI ships the `openai` CLI. Cloudflare ships `wrangler`.

The CLI is the natural entry point for developer agents. EvanCore's product surface is `evan ...`. Not an SDK. Not a web console. Not a dashboard. The SDK is a byproduct of the CLI; the web console, if it ever exists, is a byproduct of the CLI.

This is deliberate. Putting the CLI first forces the product to be scriptable, diff-able, CI-friendly, and stateless-looking — which are the same properties that make Terraform / kubectl / docker durable across generations of frontends.

---

## 4. MCP is a Tool backend, not a product layer

The biggest category mistake people make about EvanCore is assuming that because it talks to MCP, it *is* an MCP platform.

MCP is an open standard for connecting agents to external systems. Anthropic's own framing calls it an "integration substrate." Microsoft Agent Framework puts MCP in the cross-runtime interop layer. **MCP is infrastructure, not a user entry point.**

In EvanCore, MCP lives inside the four-line open foundation:

| Line | What it does | Backends |
|---|---|---|
| Role | Resolves role identity | `builtin://`, custom |
| Skill | Resolves skill manifests (SKILL.md) | `builtin://`, `fs://` |
| Memory | Resolves memory backend | `builtin://`, future |
| **Tool** | Resolves tool backend | `builtin://`, **`mcp://`** |

MCP is *one* scheme under *one* line. When we talk about "MCP Phase 2" internally, we mean hardening the `mcp://` backend (persistent session, reconnect-once, per-call escape hatch) — a local hardening of a local backend. Not a platform pivot.

If you strip MCP out of EvanCore, EvanCore's differentiator is unchanged: lifecycle for agent harnesses. If you strip lifecycle out, what remains is just a thin MCP client — nothing worth building a CLI for.

---

## 5. Closed core, open verification

The EvanCore engine — assembler, registry federation, signing kernel, trust resolution, revocation checker, the four-line runtime — is not in this repository.

What *is* in this repository:

- The spec schema (so you can reason about what a harness *is*)
- The example harness specs (so you can see what a deliverable looks like)
- The evaluation log (so you can see which external projects were considered and why they were or weren't absorbed)
- The philosophy (this file)
- The lifecycle visualization (`docs/index.html`)

This is the same three-layer strategy used by the sibling project [`memory-core-eval`](https://github.com/Evanyuan-builder/memory-core-eval): **open verification · semi-open interface · closed core**. The scorecard is public; the engine is not.

Numbers over marketing. 607 tests are real. The assembler behind them is not yours to read, but it's not the reason you'd adopt EvanCore anyway — the reason is the lifecycle shape, and that's public.

---

## 6. What EvanCore will never become

Anti-goals are load-bearing. EvanCore will not:

- **Add a runtime.** There is no plan to execute agent steps. LangGraph and CrewAI already do this well.
- **Add orchestration primitives.** There is no plan to model conditional flows, retries, branching. That's a framework concern.
- **Become an MCP registry.** Public MCP registries exist (`modelcontextprotocol.io/servers`, community curations). EvanCore consumes MCP; it doesn't catalogue it.
- **Make Harness or Team independently releasable.** The harness *is* the atomic release unit. Team members inside a harness are not separately versioned.
- **Collapse `assemble` into `release`.** `assemble` produces an archive-in-place package; `release` produces a content-addressed artifact. They are different operations because they have different guarantees.

These are the same anti-goals whether the project is 607 tests or 60,700 tests. The lifecycle shape is the product.
