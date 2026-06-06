# `evan compare` — the lab

Two configs, one task, an LLM judge.

```bash
evan compare variant_a.yaml variant_b.yaml \
  --trigger "Write a 3-sentence pitch for a tool that version-controls AI agent configs." \
  --criteria "punchy, concrete, no buzzwords"
```

- `base.yaml` is the shared config. `variant_a` / `variant_b` are overlays
  that change only the model — change anything (prompt, tools, memory,
  temperature, routing) the same way and compare again.
- `provider: claude-code` runs on your local `claude` CLI with **no API key**.
  Swap to `ollama` for offline, or `anthropic` / `openai` for raw-API costs.
- The winner ships with `evan release variant_b.yaml`.

> The engine that runs the comparison is private; these specs and the rendered
> transcript in the top-level README are the public surface.
