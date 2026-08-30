# Exact-panel screenshot fallback

> **Refresh note (2026-08-28):** mission-era panel names are stripped; the exact-panel
> principle is unchanged and current. Live panels to name in its place: roster, chat
> transcript, board/kanban, office scene, agent console.

Use this when validating Launcher UI changes with Stage C MCP screenshot QA.

## Durable lesson

The visual proof standard is a real PNG path showing the exact requested panel. However,
the semantic MCP tab enum/button registry can lag behind app surfaces and omit a direct
route to a page while still exposing a nearby shell tab. Do not coordinate-click or invent
unsupported semantic actions just to force the exact panel into frame.

## Safe fallback pattern

1. Use the strongest available semantic route first (`open_app_tab`, `set_tab`,
   `get_buttons`, `click_button`, `shell.scroll.*`).
2. If the requested exact page/panel is not exposed by the MCP schema or button registry,
   do **not** claim exact-panel screenshot QA.
3. Capture the nearest supported shell/surface screenshot only as partial evidence.
4. Pair it with deterministic UI tests / build / analyze that cover the exact panel.
5. Report the limitation explicitly:

```text
Live Stage C shell screenshot captured, but exact <panel> screenshot QA was not performed
because the current MCP semantic surface does not expose a direct route/control for <panel>.
```

This preserves the screenshot-quality bar without adding coordinate-click flakiness or a
new helper path.

## Do not capture as durable failure

Do not write "screenshot QA is unavailable for this panel" as a permanent rule. Treat it
as a semantic-surface coverage gap for the current build/session, and verify again after
MCP control schemas are rebuilt or expanded.
