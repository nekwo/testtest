# Launcher Library semantic MCP item/navigation controls

## Capability now expected

Stage C Launcher MCP Library controls should expose real page-local Library semantics, not only global navigation or `library.view_mode` toggles.

Validated shape:
- `get_buttons(scope=library.item)` returns real visible Library items/cards/rows from the app's Library state.
- Item ids are deterministic and safe, shaped like `library.item.<safe_id>`, e.g. `library.item.external:steam:1068820`.
- Item records include safe label, `scope`, `kind=item`, `order`, `enabled`, `visible`, and `selected` state.
- `get_buttons(scope=library.navigation)` returns directional controls:
  - `library.nav.left`
  - `library.nav.right`
  - `library.nav.up`
  - `library.nav.down`
- `click_button(id=<library.item...>)` semantically selects/focuses the item; follow up with `get_buttons(scope=library.item)` to verify `selected=true`.
- Do not assume a second item click expands/details-opens the selected Library card. In the live MCP contract, item click is focus/select; expansion/details is exposed as the separate semantic action `library.action.open_details` with label shaped like `Open details for selected item`.
- After clicking `library.action.open_details`, verify `mcp_launcher_qa_get_widget_state(widget=library.details)` reports `mounted=true`, `details_open=true`, `details_item_id_safe=<selected item>`, and `selected_matches_details=true`, then capture a screenshot path as visual evidence.
- `click_button(id=library.nav.right)` / left/up/down changes Library selection/focus semantically; verify via another `get_buttons(scope=library.item)` and capture screenshot proof.

## Validated operator smoke

1. `mcp_launcher_qa_open_app_tab(tab=library, browser_login=true, credential_profile=stagec-smoke, screenshot=false, reap_stale=true)`.
2. `mcp_launcher_qa_get_buttons(scope=library.item)`.
3. Select first visible item by id, e.g. OVR Toolkit.
4. Capture screenshot with `mcp_launcher_qa_screenshot_window`.
5. Click `library.nav.right` repeatedly.
6. Verify selected item changes after each click.
7. Capture second screenshot.
8. For details/expansion, click `library.action.open_details` rather than clicking the selected item again.
9. Verify `library.details` state (`mounted=true`, `details_open=true`, `selected_matches_details=true`) and capture a details-open screenshot path.

Example observed sequence:
- first selected: `OVR Toolkit`
- right 1: `Death end re;Quest`
- right 2: `Skyrim Script Extender (SKSE)`
- right 3: `Wallpaper Engine`
- right 4: `Beat Saber`
- right 5: `Phantasy Star Online 2 New Genesis`

## Pitfalls

- Parse MCP helper envelopes through `response_safe.data.buttons`; top-level `buttons` may not exist.
- A fresh helper `kill_launcher` may return `app_not_attached`; if this test launched the app and cleanup is required, use a bounded process cleanup after recording evidence.
- Do not claim item selection from click envelopes alone. Always verify selected/focused/current state with `get_buttons` or a compact state probe, then screenshot when visual behavior matters.
- Do not treat `button_click_failed` from a repeat click on the already-selected Library item as proof that details/expansion is broken. Check whether the semantic details action exists (`library.action.open_details`) and use that action for expansion.
- Keep coordinate clicks and SharedPreferences mutation out of acceptance proof.
