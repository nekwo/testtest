# Stage C MCP Kanban efficiency lessons — 2026-05-18

Use this reference when Stage C MCP work is being routed through Kanban, especially implementation → QA chains.

## Durable lessons

- Prefer semantic MCP evidence over long screenshot retries. If live MCP state/buttons prove the behavior and `screenshot_window` returns repeated low-information captures, classify that as a screenshot-helper/runtime artifact and route it separately instead of extending the implementation card.
- Treat Library item opening as a two-step semantic flow: select via `library.item.select.<safe-id>`, then open via `library.action.open_details`, then verify `library.details` state.
- If visible page-local tabs/subtabs exist but are not registered in `get_buttons()`, create a semantic-control gap card rather than coordinate-clicking.
- For Stage C MCP implementation cards, avoid duplicate test/analyze loops after compaction. Reuse prior green evidence when command, exit code, timestamp/tree state, and artifact path are present; rerun only after code changes, missing evidence, or a failed suite.
- For Stage C MCP cards that rebuild the MCP server EXE, preflight stale/locked binaries before compile or compile to a temp path and replace only when safe.

## Process-efficiency checklist

1. Start with `open_app_tab` / `get_buttons` / `click_button` / `get_widget_state` semantic proof.
2. Use verb-explicit controls; do not normalize ambiguous IDs such as selector controls named like open actions.
3. Run targeted Flutter/MCP tests once after implementation.
4. Capture screenshot once for user-facing evidence; retry once if needed.
5. If semantic state passes but screenshot helper fails repeatedly, create a helper bug/process note and let QA own visual acceptance.
6. Hand off to QA with exact commands, exit codes, state fields, and artifact paths so compaction does not trigger wasteful reruns.
