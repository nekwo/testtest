# Launcher Library semantic MCP item/navigation controls

> **Edit note (2026-08-28):** this file previously taught the ambiguous id shape
> `library.item.<safe_id>` with `kind=item`. That shape is a **contract defect**, not a
> contract — see `stagec-library-select-vs-open-contract.md` and
> `stagec-semantic-control-naming-and-page-local-tabs.md`. The verb-explicit selector
> shape is authoritative; this file now defers to it and keeps only the navigation /
> operator-smoke material that is unique to it.

## Capability now expected

Stage C Launcher MCP Library controls should expose real page-local Library semantics,
not only global navigation or `library.view_mode` toggles.

Validated shape:

- `get_buttons(scope=library.item)` returns the real visible Library items/cards/rows
  from the app's Library state.
- **Item ids are verb-explicit selectors.** The authoritative shape is
  `library.item.select.<safe-id>`, scope `library.item`, kind `select`, label
  `Select <title>` — see `stagec-library-select-vs-open-contract.md`. If the live schema
  still emits `library.item.<safe-id>` with `kind=item`, treat the running app/MCP binary
  as stale (rebuild and relaunch) or file the naming defect; do not normalize the
  ambiguous shape into your workflow.
- Item records include a safe label, `scope`, `kind`, `order`, `enabled`, `visible`, and
  `selected` state.
- `get_buttons(scope=library.navigation)` returns directional controls:
  - `library.nav.left`
  - `library.nav.right`
  - `library.nav.up`
  - `library.nav.down`
- Clicking a `library.item.select.*` id semantically selects/focuses the item; follow up
  with `get_buttons(scope=library.item)` to verify `selected=true`.
- A second click on a selected item does **not** expand/open details. Expansion is the
  separate semantic action `library.action.open_details`, label shaped like
  `Open details for selected item`.
- After clicking `library.action.open_details`, verify
  `mcp_launcher_qa_get_widget_state(widget=library.details)` reports `mounted=true`,
  `details_open=true`, `details_item_id_safe=<selected item>`, and
  `selected_matches_details=true`, then capture a screenshot path as visual evidence.
- `click_button(id=library.nav.right)` / left/up/down changes Library selection/focus
  semantically; verify via another `get_buttons(scope=library.item)` and capture
  screenshot proof.

## Validated operator smoke

1. `mcp_launcher_qa_open_app_tab(tab=library, browser_login=true, credential_profile=stagec-smoke, screenshot=false, reap_stale=false)`.
2. `mcp_launcher_qa_get_buttons(scope=library.item)`.
3. Select the first visible item by its `library.item.select.*` id, e.g. OVR Toolkit.
4. Capture a screenshot with `mcp_launcher_qa_screenshot_window`.
5. Click `library.nav.right` repeatedly.
6. Verify the selected item changes after each click.
7. Capture a second screenshot.
8. For details/expansion, click `library.action.open_details` rather than clicking the
   selected item again.
9. Verify `library.details` state (`mounted=true`, `details_open=true`,
   `selected_matches_details=true`) and capture a details-open screenshot path.

Example observed navigation sequence:

- first selected: `OVR Toolkit`
- right 1: `Death end re;Quest`
- right 2: `Skyrim Script Extender (SKSE)`
- right 3: `Wallpaper Engine`
- right 4: `Beat Saber`
- right 5: `Phantasy Star Online 2 New Genesis`

## Pitfalls

- Parse MCP helper envelopes through `response_safe.data.buttons`; a top-level `buttons`
  key may not exist.
- A fresh helper `kill_launcher` may return `app_not_attached`; if this run launched the
  app and cleanup is required, use a bounded process cleanup after recording evidence.
- Do not claim item selection from click envelopes alone. Always verify
  selected/focused/current state with `get_buttons` or a compact state probe, then
  screenshot when visual behavior matters.
- Do not treat `button_click_failed` from a repeat click on the already-selected Library
  item as proof that details/expansion is broken. Use `library.action.open_details`.
- Keep coordinate clicks and SharedPreferences mutation out of acceptance proof.
