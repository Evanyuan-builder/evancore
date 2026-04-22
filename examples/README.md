# Examples

Runnable starting points for the EvanCore harness spec schema. All three files validate against the public schema in [`../docs/spec-schema.md`](../docs/spec-schema.md).

| File | Kind | Purpose |
|---|---|---|
| [`agent-harness.yaml`](agent-harness.yaml) | `AgentHarness` | Single-agent assembly exercising all four backend lines (Role / Skill / Memory / Tool) |
| [`team-harness.yaml`](team-harness.yaml) | `TeamHarness` | Three-member research team, sequential routing, shared memory namespace |
| [`trust.yaml`](trust.yaml) | trust file | Per-host Ed25519 key trust + revocation policy for `evan verify` / `evan pull --verify` |

These specs are the *inputs* to the EvanCore delivery layer. The *outputs* — content-addressed signed artifacts — are produced by the (closed) EvanCore engine when you run `evan release`.

The schemas shown here are stable and public. The assembly + signing kernel that consumes them is closed. See [`../README.md`](../README.md) and [`../docs/philosophy.md`](../docs/philosophy.md) for why.
