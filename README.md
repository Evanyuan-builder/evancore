<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=180&section=header&text=EvanCore&fontSize=55&fontColor=ffffff&animation=fadeIn&fontAlignY=38&desc=Terraform%20for%20AI%20agents&descAlignY=60&descSize=16" width="100%"/>

<a href="https://github.com/Evanyuan-builder/evancore">
  <img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=500&size=18&duration=3500&pause=800&color=6C7B95&center=true&vCenter=true&width=700&lines=content-addressed;signed;reversible" />
</a>

<p>
  <img src="https://img.shields.io/badge/focus-Harness%20Engineering-6C7B95?style=flat-square" />
  <img src="https://img.shields.io/badge/surface-CLI--first-6C7B95?style=flat-square" />
  <img src="https://img.shields.io/badge/tests-607%20passing-6C7B95?style=flat-square" />
  <img src="https://img.shields.io/badge/status-private%20beta-6C7B95?style=flat-square" />
  <img src="https://img.shields.io/badge/license-Apache--2.0-6C7B95?style=flat-square" />
</p>

<a href="https://evanyuan-builder.github.io/evancore/">
  <img src="https://img.shields.io/badge/▶%20live%20demo-evanyuan--builder.github.io%2Fevancore-7cc4d4?style=for-the-badge&labelColor=0a0d12" />
</a>

</div>

---

**CLI-first Harness Engineering — content-addressed, signed, reversible.**

Most agent tooling focuses either on runtime capabilities or orchestration. EvanCore focuses on the delivery layer in between: how an agent harness gets assembled, versioned, signed, released, rolled back, upgraded, and promoted across environments — the way Terraform handles infrastructure, or the way container registries handle images.

The engine is closed. The design, the spec schema, the philosophy, and the evaluation log are open.

---

## What this is / is not

**Is:**
- A CLI-first **delivery layer** for agent harnesses (`evan assemble / release / rollback / upgrade / promote / publish / pull / verify`)
- **Content-addressed** artifacts with deterministic hashing (blake2b-64 prefix)
- **Signed** releases with Ed25519 + per-host trust files + revocation v2
- **Reversible** lifecycle — every release has an inverse (rollback / upgrade target / promote source)
- An **open foundation** for four backend lines (Role / Skill / Memory / Tool), where MCP is one Tool-line backend — not the center of gravity

**Is not:**
- An **agent framework** (no runtime, no orchestration primitives, no LangGraph/CrewAI overlap)
- A **hosting platform** (you still choose where your agents run)
- A **pure MCP registry** — MCP is *one* Tool backend scheme, not the product

---

## The three layers

| Layer | Role | Not this |
|---|---|---|
| **CLI** | Product surface. `evan assemble / release / promote / rollback / upgrade / diff / validate / publish / pull / keys / ...` | Not an SDK |
| **Harness lifecycle** | The actual differentiator. spec → assemble → release → promote → rollback → upgrade; content-addressed; signed; revocable | Not a runtime |
| **MCP / Tool backends** | One integration substrate among several (`mcp://`, `builtin`, …) behind the Tool line | Not the user entry point |

> Anthropic positions Claude Code as "an agentic coding tool that lives in your terminal." Microsoft's Agent Framework 1.0 does the same with Azure Developer CLI. **The CLI is where developers meet the agent stack.** MCP is the integration substrate *underneath*, not the surface on top. EvanCore agrees.

---

## Core operations (abbreviated)

```bash
# define
$ evan validate app.yaml
✓ app.yaml passes schema + reference checks

# release
$ evan release app.yaml --sign alice@acme
📦 SPEC_RELEASED       h-abc123f7d9e2
✍  SPEC_SIGNED         alice@acme
🚀 HARNESS_DEPLOYED    app (av)

# reverse
$ evan rollback h-abc123 --yes
↩  HARNESS_ROLLED_BACK h-3c21b0f8e5a4

# promote across envs
$ evan promote prod.yaml --from staging-app
🚀 HARNESS_DEPLOYED    prod-app (promoted)

# distribute
$ evan publish h-abc123 --registry acme
$ evan pull      acme/app@h-abc123 --verify
✓ signature verified · trust: alice@acme
```

See [`examples/`](examples/) for the example harness spec that produces the artifact above.

---

## What's open here / what's closed

This repository is the **public surface** of EvanCore:

| Open (this repo) | Closed (private engine) |
|---|---|
| `docs/index.html` — interactive lifecycle visualization | Assembler (spec → harness packaging) |
| `docs/philosophy.md` — why lifecycle, not runtime | Release machinery (content-addressed storage, hash chain) |
| `docs/evaluations.md` — ecosystem evaluation log | Signing / trust / revocation kernel |
| `docs/spec-schema.md` — public harness spec schema | Registry federation (file:// + https://) |
| `examples/*.yaml` — runnable example specs | Assemble/release/rollback/upgrade/promote/publish CLI impl |
| `LICENSE` — Apache-2.0 (on materials in this repo) | v2 open-foundation four-line runtime (Role/Skill/Memory/Tool) |

**Ship status:** v1 A–G + v2 four-line foundation + release closure trilogy + B3 Phase A/A.1/B/B.1/B.2 + MCP Phase 2 (persistent session, reconnect-once, per-call escape hatch) — **607 tests passing** as of 2026-04-22.

The verification shape (schema, evaluation transparency, signed artifacts) is here. The engine isn't.

---

## Positioning (honest)

| If you want… | Use this instead |
|---|---|
| A runtime that executes agent steps | LangGraph, CrewAI, Microsoft Agent Framework |
| A hosted platform for running agents | Vercel AI, Modal, Fly.io Machines |
| A tool-use protocol / integration substrate | MCP itself — EvanCore consumes it, doesn't replace it |
| A spec-driven coding workflow | Spec Kit, GSD (different "spec" — they mean prompt specs for code generation) |

EvanCore sits **between** the spec file and the running agent. If your harness spec is the source of truth, EvanCore is how that source of truth becomes a signed, reversible, distributable artifact.

---

## Evaluation transparency

Every external project we evaluated for absorption — and every one we **declined** — is logged in [`docs/evaluations.md`](docs/evaluations.md). The default verdict is *decline* or *take idea only*; absorbing code is rare and requires a specific gap hit. This log exists so the project's scope stays defensible in public.

---

## Access

- **Design, schema, examples, evaluation log:** this repo (Apache-2.0)
- **Engine / CLI binary:** private beta — open an issue or email to request access
- **Visual tour:** open [`docs/index.html`](docs/index.html) in a browser

---

## Related work

- [`memory-core-eval`](https://github.com/Evanyuan-builder/memory-core-eval) — sibling project, open verification layer for agent memory systems
- [`Evanyuan-builder`](https://github.com/Evanyuan-builder) — profile, three-layer strategy (*open verification · semi-open interface · closed core*)

---

## License

Apache-2.0 on materials in this repository (docs, schema, examples, landing page). See `LICENSE`.

The EvanCore engine is **not** covered by this license and is not distributed here.
