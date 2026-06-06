<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=180&section=header&text=EvanCore&fontSize=55&fontColor=ffffff&animation=fadeIn&fontAlignY=38&desc=A%20lab%20for%20AI%20agent%20harnesses&descAlignY=60&descSize=16" width="100%"/>

<a href="https://github.com/Evanyuan-builder/evancore">
  <img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=500&size=18&duration=3500&pause=800&color=6C7B95&center=true&vCenter=true&width=700&lines=run+two+configs;judge+which+won;ship+the+winner" />
</a>

<p>
  <img src="https://img.shields.io/badge/focus-Harness%20Lab-6C7B95?style=flat-square" />
  <img src="https://img.shields.io/badge/surface-CLI--first-6C7B95?style=flat-square" />
  <img src="https://img.shields.io/badge/run-no%20API%20key-7cc4d4?style=flat-square" />
  <img src="https://img.shields.io/badge/tests-633%20passing-6C7B95?style=flat-square" />
  <img src="https://img.shields.io/badge/license-Apache--2.0-6C7B95?style=flat-square" />
</p>

<a href="https://evanyuan-builder.github.io/evancore/">
  <img src="https://img.shields.io/badge/▶%20live%20demo-evanyuan--builder.github.io%2Fevancore-7cc4d4?style=for-the-badge&labelColor=0a0d12" />
</a>

</div>

---

**Change an agent config, instantly know if it got better.**

You tune an agent — a prompt, a tool, a memory setting, a temperature — and you don't actually know if it improved. Last week's good config is gone. Two setups can only be compared by gut. Your agent's configuration lives in scattered prompt strings, a Notion doc, and someone's head.

EvanCore is the lab that fixes that: **run two harness configs on the same task, score the outcomes, and see which won** — output, config diff, cost, verdict. And because every config is a content-addressed, signed, reversible artifact, the winner ships with one command.

The engine is closed. The design, the spec schema, the philosophy, and the evaluation log are open.

---

## The lab: `evan compare`

Two configs differing only by model. Same task. The judge decides.

```console
$ evan compare variant_a.yaml variant_b.yaml \
    --trigger "Write a 3-sentence pitch for a tool that version-controls AI agent configs." \
    --criteria "punchy, concrete, no buzzwords"

╭──────────────────────────────╮ ╭──────────────────────────────────╮
│ Variant A · variant_a.yaml   │ │ Variant B · variant_b.yaml ◀ winner│
│ score   7.0/10               │ │ score   9.0/10                     │
│ cost    $0.0020              │ │ cost    $0.0023                    │
│ tokens  10 in / 509 out      │ │ tokens  3 in / 152 out             │
│ status  ✓                    │ │ status  ✓                          │
╰──────────────────────────────╯ ╰──────────────────────────────────╯

Config delta (A → B)
  ~ model.model_id: 'haiku' → 'sonnet'

🏆 Variant B wins — 9.0 vs 7.0
B opens with a sharper pain point, the 'last Tuesday' detail is concrete,
and 'git for the part of your stack that lives in someone's head or a
Notion doc' lands harder than A's generic closer.

Ship it:  evan release variant_b.yaml
```

Change anything — prompt, tools, memory backend, temperature, routing — author it as an overlay, and compare again. That loop *is* the lab. The judge is an LLM scored on your stated criteria; the cost and token numbers are real; the config delta is computed from the specs.

See [`examples/compare/`](examples/compare/) for the variant specs that produce this.

---

## Run it with no API key

EvanCore harnesses can run on the **local `claude` CLI** (Claude Code) — using the machine's existing auth, no `ANTHROPIC_API_KEY` required. If you have Claude Code installed, you can compare real Claude models out of the box:

```yaml
model:
  provider: claude-code   # routes through your local `claude` CLI
  model_id: sonnet        # or haiku / opus / a full model id
```

```console
$ evan compare a.yaml b.yaml -t "..." --judge "claude-code:sonnet"
```

Ollama (`provider: ollama`) works the same way for fully-offline runs. Native `anthropic` / `openai` providers are there when you want raw-API cost accounting.

---

## The spine beneath the lab

A comparison is only worth trusting if the thing you compared is reproducible. EvanCore's delivery layer is what makes that true — and what turns "B won" into "B is shipped."

| Capability | What it buys the lab |
|---|---|
| **Content-addressed specs** (blake2b-64) | The exact config you scored is the exact config you ship — no drift |
| **`evan diff`** | The config delta in every comparison |
| **`evan release / rollback / upgrade / promote`** | Ship the winner, reverse it, move it across environments — every release has an inverse |
| **Signed artifacts** (Ed25519 + trust + revocation) | Distribute a winning config others can verify |
| **Overlays** (Kustomize-style) | Author A/B/C variants without copy-pasting whole specs |

```bash
# the winner becomes a signed, reversible artifact
$ evan release variant_b.yaml --sign alice@acme
📦 SPEC_RELEASED    h-abc123f7d9e2
✍  SPEC_SIGNED      alice@acme
🚀 HARNESS_DEPLOYED app (av)

# changed your mind after the next experiment?
$ evan rollback h-abc123 --yes
↩  HARNESS_ROLLED_BACK
```

---

## What this is / is not

**Is:**
- A **lab** for agent harnesses — run two configs on one task, judge which won (`evan compare`)
- A **delivery spine** that makes the winner reproducible and shippable (assemble / diff / release / rollback / upgrade / promote / publish / pull / verify)
- **Backend-agnostic** runs — `claude-code` (no key), `ollama` (offline), `anthropic`, `openai`

**Is not:**
- An **agent framework** (no runtime DSL, no orchestration primitives, no LangGraph/CrewAI overlap)
- A **hosting platform** (you still choose where agents run)
- A **tracing/observability** product — those watch *runs*; EvanCore version-controls and compares the *config* that produced them

---

## What's open here / what's closed

This repository is the **public surface** of EvanCore:

| Open (this repo) | Closed (private engine) |
|---|---|
| `docs/index.html` — interactive lifecycle visualization | Compare loop (run → judge → side-by-side) |
| `docs/philosophy.md` — why a lab, not a runtime | Assembler (spec → harness packaging) |
| `docs/evaluations.md` — ecosystem evaluation log | Release machinery (content-addressed storage, hash chain) |
| `docs/spec-schema.md` — public harness spec schema | Signing / trust / revocation kernel |
| `examples/*.yaml` — runnable example specs | Registry federation (file:// + https://) |
| `LICENSE` — Apache-2.0 (on materials in this repo) | CLI implementation + v2 four-line backend runtime |

**Ship status:** v1 A–G + v2 four-line foundation + release closure trilogy + B3 supply-chain (A/A.1/B/B.1/B.2) + MCP Phase 2 + the **lab** (`evan compare`, LLM-judge, `claude-code` backend) — **633 tests passing** as of 2026-06-06.

The verification shape (schema, evaluation transparency, signed artifacts, the real comparison transcript above) is here. The engine isn't.

---

## Positioning (honest)

| If you want… | Use this instead |
|---|---|
| A runtime that executes agent steps | LangGraph, CrewAI, Microsoft Agent Framework |
| A hosted platform for running agents | Vercel AI, Modal, Fly.io Machines |
| Tracing / eval dashboards over agent *runs* | LangSmith, Langfuse, Braintrust |
| A tool-use protocol / integration substrate | MCP itself — EvanCore consumes it, doesn't replace it |

The difference from the tracing tools: they observe the *run*. EvanCore version-controls the *config* and tells you which config to keep. If your harness spec is the source of truth, EvanCore is the lab bench it sits on and the press that ships it.

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
