# Mission Control exact-panel screenshot fallback

Use this when validating Mission Control / Agent Runtime Harness UI changes with Stage C MCP screenshot QA.

## Durable lesson

The visual proof standard remains a real PNG path. However, the semantic MCP tab enum/buttons can lag behind app surfaces or omit a direct page such as Mission Control while still exposing a nearby shell tab (for example `ai`). Do not coordinate-click or invent unsupported semantic actions just to force the exact panel.

## Safe fallback pattern

1. Use the strongest available semantic route first (`open_app_tab`, `set_tab`, `get_buttons`, `click_button`).
2. If the requested exact page/panel is not exposed by the MCP schema or button registry, do **not** claim exact-panel screenshot QA.
3. Capture the nearest supported shell/surface screenshot only as partial evidence.
4. Pair it with deterministic UI tests/build/analyze that cover the exact panel.
5. Report the limitation explicitly:

```text
Live Stage C shell screenshot captured, but exact <panel> screenshot QA was not performed because the current MCP semantic surface does not expose a direct route/control for <panel>.
```

This preserves the screenshot-quality bar without adding coordinate-click flakiness or a new helper path.

## Do not capture as durable failure

Do not write “Mission Control screenshot QA is unavailable” as a permanent rule. Treat it as a semantic-surface coverage gap for the current build/session, and verify again after MCP control schemas are rebuilt or expanded.
