# Stage C MCP Library item selection gap — 2026-05-18

## Trigger

Tony asked whether Alice could select one of the Launcher Library items with MCP after the MCP-first screenshot flow was validated.

## What was verified

The MCP-first launch/navigation path worked:

- `mcp_launcher_qa_open_app_tab` with `tab=library`, `browser_login=true`, `credential_profile=stagec-smoke`, and `reap_stale=false` returned `ok=true`.
- The app was attached and authenticated.
- `navigation_state.selected_tab=library`.
- The Launcher window and QA control server were live.

The live tool catalog exposed:

- `mcp_launcher_qa_get_buttons`
- `mcp_launcher_qa_click_button`
- `mcp_launcher_qa_screenshot_window`

But the semantic button schema only included scopes:

- `shell.nav`
- `blocking_modal`
- `library.view_mode`

It did **not** expose a Library item/game/app scope such as `library.item`.

## Important boundary

Through the bounded one-shot helper, low-level interactive calls after `open_app_tab` failed with `app_not_attached`:

```text
failure_class: app_not_attached
message_safe: Launcher is not attached. Call mcp_launcher_qa_launch_or_attach or mcp_launcher_qa_open_app_tab first.
```

This happened because the helper starts a fresh MCP server lifetime per call. It is still an MCP path, but it is not a persistent multi-call native MCP session. Do not treat this as proof that the app cannot be controlled; treat it as a helper/session-boundary limitation.

## Correct conclusion

Do not claim that Library item selection works via MCP until both are true:

1. Alice/worker has a persistent native MCP session/tool bridge where `open_app_tab → get_buttons → click_button → screenshot_window` all run against the same attached app.
2. The Launcher registers page-local Library item controls, e.g. scope `library.item` with stable ids and safe labels for visible games/apps.

Until then, supported MCP actions are tab navigation, auth/open, screenshots, and whatever registered controls are in the schema (currently view-mode controls, not Library item selection).

## Desired follow-up capability

Add Library item semantic registry support:

- Scope: `library.item`.
- Stable ids: e.g. `library.item.<slug-or-content-id>`.
- Safe labels: display names without secrets/paths/tokens.
- State fields: `visible`, `enabled`, maybe `selected`/`focused` if applicable.
- Click behavior: selecting/focusing an item, failing closed on not found/ambiguous/disabled/invisible.
- Verification: after click, query state/buttons again and capture `mcp_launcher_qa_screenshot_window` proof.

Avoid coordinate clicks as acceptance evidence for this class of task.
