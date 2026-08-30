# Semantic shell scroll and below-fold screenshot proof

> **Refresh note (2026-08-28):** the original targeted the Live Agent Terminal, which no
> longer exists. The `shell.scroll.*` contract and the scroll-then-verify discipline are
> the durable content and apply to any below-fold Launcher surface. This file is now the
> single home for the `shell.scroll` contract.

## When to use

Use this when the operator asks for a screenshot of a panel that is below the fold, or
whenever a page section must be brought into frame and the approved path must avoid
coordinate scrolling.

## The `shell.scroll` contract

- `get_buttons(scope="shell.scroll")` should expose semantic scroll controls for the
  active shell content root:
  - `shell.scroll.up`
  - `shell.scroll.down`
  - `shell.scroll.top`
  - `shell.scroll.bottom`
- Controls are `kind: scroll`, `scope: shell.scroll`.
- The control/state envelope should include safe position telemetry: `y`, `max_y`,
  `viewport_height`, `can_scroll_up`, `can_scroll_down`.
- `click_button(id="shell.scroll.down")` animates the active content scrollable; the
  follow-up state/buttons read should show a changed `y` or changed `can_scroll_*` values.

If `get_buttons(scope="shell.scroll")` is missing on a content-rich shell page, classify
that as a Stage C semantic-control regression/gap. Do not fall back to coordinate
scrolling as if it were acceptable evidence.

## Durable workflow

1. Launch/open the target surface through Stage C MCP with the pinned Harness smoke env
   when relevant:
   - `profile: stagec-smoke`
   - `credential_profile: stagec-smoke`
   - `browser_login: true` when auth may be needed
   - `hermes_profile: alice`
   - `harness_runtime_root` and `hermes_home` explicitly pinned for parity screenshots.
2. Confirm semantic scroll affordances before scrolling:
   - call `get_buttons(include_disabled=true, include_invisible=true)`
   - require `shell.scroll.down` / `shell.scroll.up` in scope `shell.scroll`
   - inspect the safe `state` map (`y`, `max_y`, `viewport_height`, `can_scroll_down`).
3. Scroll semantically, not by coordinates:
   - preferred: `click_button(id="shell.scroll.down")` / `shell.scroll.bottom`
   - alternate: `scroll(target="shell", direction="down", amount="page")` when the live
     app exposes the same observer path.
4. Verify movement before screenshot:
   - re-read `get_buttons(...)`
   - confirm `state.y` increased and `can_scroll_up` changed as expected.
5. Capture visual proof with `open_app_tab(... screenshot=true, force_relaunch=false,
   reap_stale=false)` or the primitive screenshot tool as appropriate.
   - Preserve the current running session; do not relaunch after scrolling unless you are
     explicitly resetting or rebuilding.
   - If PrintWindow blanks after semantic gates pass, accept the Marionette internal
     Flutter screenshot fallback and label it honestly.
6. For delivery, send the PNG immediately with `MEDIA:<path>` plus only a compact note
   about scroll verification if useful.

## Page-local scroll gap

Generic `shell.scroll` moves the active page/content column. It may not reach content
inside a page-local panel that owns its own scrollable. If `shell.scroll.bottom` reaches
`y=max_y` and the requested panel content is still not in frame, that is a **page-local
semantic-scroll gap**: report it, pair the nearest live PNG with deterministic widget
tests, and do not coordinate-scroll or overclaim exact-panel pixels.

## Fresh-build pitfall

If `shell.scroll` controls are absent in a fresh-looking session, the running Launcher may
still be an older binary or built against the wrong target.

Recovery:

1. Kill/close only the running Launcher process if it locks `WebView2Loader.dll`.
2. Rebuild the QA target, not the regular target:
   - `flutter build windows --debug --target lib/main_marionette.dart`
3. Relaunch through Stage C MCP and re-check `get_buttons` for `shell.scroll.*` controls.

Stage C correctly rejects a regular `flutter build windows --debug` binary with
`launch_wrong_debug_target_missing_marionette`; treat that as a freshness/build-target
guardrail, not a product bug.

## Interpretation

A visible panel header with no rows is still valid visual proof of the panel area, but it
does not prove populated content. Report the true state, e.g. `no redaction-safe events
have been recorded yet`. The product contract for any operator transcript remains a
redaction-safe structured view, not raw hidden chain-of-thought or unsafe logs.
