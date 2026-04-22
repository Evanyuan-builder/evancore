# Harness Spec Schema

> The shape of a harness spec. This is the source of truth that `evan assemble`, `evan release`, `evan rollback`, `evan promote`, and `evan upgrade` operate on.

A harness spec is a YAML document that declares the complete assembly of an agent runtime — role, memory, model, tools, skills, policies, and recovery behavior — without specifying *how* it runs. The spec is the **input** to the delivery layer; the artifact that comes out of `evan release` is content-addressed and signed.

---

## Two kinds

EvanCore specs declare one of two kinds:

| `kind` | What it is | When to use |
|---|---|---|
| `AgentHarness` | A single agent assembly | One configured agent with its own role, memory, tools |
| `TeamHarness` | A coordinated multi-member assembly | Multiple members with shared memory and a routing policy |

Both share a common release/rollback/promote lifecycle. Team members are **not** independently releasable — the team is the atomic release unit. (This is a deliberate invariant, see [`philosophy.md`](philosophy.md) §6.)

---

## `AgentHarness` shape

```yaml
kind: AgentHarness
id: my-agent                             # stable identity within a registry namespace

# -- who this agent is (resolved via Role backend) --
role:
  ref: "builtin://roles/researcher"      # scheme:// dispatch to a Role backend
  version: "latest"                      # or a pinned version

# -- what it remembers (resolved via Memory backend) --
memory:
  private_ns: "my-agent.private"         # private namespace (required)
  shared_ns: []                          # optional shared namespaces
  ttl: "30d"                             # memory retention

# -- what model it uses --
model:
  provider: ollama                       # or openai, anthropic, azure, ...
  model_id: qwen3:8b
  temperature: 0.3
  max_tokens: 4096

# -- what it can do (tools resolved via Tool backend) --
tools:
  - id: memory-write                     # builtin tool
    permissions: [write]
  - id: memory-read
    permissions: [read]
  - id: demo__echo                       # MCP tool: {alias}__{name}
    permissions: [read]

# -- what it knows (skills resolved via Skill backend) --
skills:
  - research-ops                         # catalog-resolved skill

# -- guardrails --
policies:
  budget_limit_usd: 1.0
  rate_limit: "100req/hour"

# -- recovery behavior --
recovery:
  max_retries: 1
  backoff: exponential
  fallback: stop
```

### Field reference

| Field | Type | Required | Notes |
|---|---|---|---|
| `kind` | enum | yes | `AgentHarness` |
| `id` | string | yes | Stable identifier; used for registry addressing |
| `role.ref` | URI | yes | `scheme://` dispatched to Role backend (`builtin://`, `rolecore://`, custom) |
| `role.version` | string | yes | `latest` or pinned |
| `memory.private_ns` | string | yes | Private namespace — can be bare (`agent.private`) or scheme-prefixed (`localmem://agent.private`) |
| `memory.shared_ns` | list | no | Shared namespaces readable by this agent |
| `memory.ttl` | duration | no | `30d`, `7d`, `1h` |
| `model.provider` | string | yes | Any supported LLM provider |
| `model.model_id` | string | yes | Provider-specific ID |
| `tools[].id` | string | yes | Tool ID; MCP tools use `{alias}__{name}` |
| `tools[].permissions` | list | yes | `read`, `write`, `exec` |
| `skills[]` | list of strings | no | Skill IDs resolved through Skill backend |
| `policies.budget_limit_usd` | number | no | Budget cap |
| `policies.rate_limit` | string | no | `<n>req/<window>` |
| `recovery.max_retries` | int | no | Default: 0 |
| `recovery.backoff` | enum | no | `linear`, `exponential`, `none` |
| `recovery.fallback` | enum | no | `stop`, `continue`, `escalate` |

---

## `TeamHarness` shape

```yaml
kind: TeamHarness
id: research-team

members:
  - id: planner
    role:      { ref: "rolecore://roles/analyst",    version: "latest" }
    model:     { provider: ollama, model_id: qwen3:14b, temperature: 0.2, max_tokens: 4096 }
    tools:     []

  - id: researcher
    role:      { ref: "rolecore://roles/researcher", version: "latest" }
    model:     { provider: ollama, model_id: qwen3:14b, temperature: 0.3, max_tokens: 4096 }
    tools:
      - { id: web-search, permissions: [read] }
      - { id: web-crawl,  permissions: [read] }

  - id: writer
    role:      { ref: "rolecore://roles/writer",     version: "latest" }
    model:     { provider: ollama, model_id: qwen3:14b, temperature: 0.3, max_tokens: 4096 }
    tools:
      - { id: report-write, permissions: [write] }
      - { id: memory-write, permissions: [write] }

routing: sequential                       # sequential | parallel | conditional

shared_memory:
  ns: "research-team.shared"
  ttl: "7d"
```

### Team-specific fields

| Field | Type | Notes |
|---|---|---|
| `members[]` | list of member objects | Each member has its own role / model / tools, but **no private memory section** — private memory is scoped per member via the team id convention |
| `routing` | enum | `sequential`, `parallel`, `conditional` |
| `shared_memory.ns` | string | Namespace visible to all members |
| `shared_memory.ttl` | duration | Retention for the shared namespace |

---

## Backend line convention

The four backend lines — Role, Skill, Memory, Tool — are the extension points of the v2 open foundation. Every reference in a spec goes through one of them.

| Line | Dispatch direction | How specs reference it |
|---|---|---|
| **Role** | forward (`scheme://` → backend) | `role.ref: "builtin://roles/X"` |
| **Skill** | forward (`scheme://` → backend) | `skills: [X]` (default `builtin://`) or explicit `fs://path` |
| **Memory** | forward (`scheme://` → backend) | `memory.private_ns: "scheme://ns"` or bare `ns` |
| **Tool** | **reverse** (tool id → backend via registry lookup) | `tools[].id: "X"` — backend resolved at assembly time |

This asymmetry — three forward-dispatched, one reverse-registered — is deliberate. Tools are dynamic (MCP servers come and go); role/skill/memory are bound at assembly. See `philosophy.md` §4 for more.

---

## Example specs

See [`examples/`](../examples/) for runnable starting points:

- [`examples/agent-harness.yaml`](../examples/agent-harness.yaml) — single agent with MCP tools
- [`examples/team-harness.yaml`](../examples/team-harness.yaml) — three-member research team, sequential routing

These specs validate against the schema even though the engine is closed — the schema itself is public.

---

## What the artifact looks like after release

When `evan release my-agent.yaml` runs, the deterministic output is a content-addressed artifact:

```
harness_id    = my-agent
content_hash  = h-abc123f7d9e2       # blake2b-64 prefix of canonical spec
signature     = Ed25519(content_hash, key=alice@acme)
trust_chain   = trust.yaml → alice@acme (valid, not revoked)
```

Every subsequent operation (`rollback`, `promote`, `pull`, `verify`) addresses the artifact by `content_hash`, not by spec file path. The spec file is the input; the artifact is the output. This is why `assemble ≠ release` — see [`philosophy.md`](philosophy.md) §6.
