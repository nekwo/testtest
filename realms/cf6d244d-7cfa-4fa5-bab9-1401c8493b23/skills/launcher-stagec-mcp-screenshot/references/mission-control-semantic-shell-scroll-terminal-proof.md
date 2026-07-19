# Mission Control semantic shell scroll + terminal screenshot proof

## When to use

Use this when Tony asks for a Mission Control terminal / right-side scene screenshot after scrolling, or when a Mission Control section is below the fold and the approved path must avoid coordinate scrolling.

## Durable workflow

1. Launch/open Mission Control through Stage C MCP with the pinned Harness smoke env when relevant:
   - `profile: stagec-smoke`
   - `credential_profile: stagec-smoke`
   - `browser_login: true` when auth may be needed
   - `hermes_profile: alice`
   - `harness_runtime_root` and `hermes_home` explicitly pinned for Harness parity screenshots.
2. Confirm semantic scroll affordances before scrolling:
   - call `get_buttons(include_disabled=true, include_invisible=true)`
   - require `shell.scroll.down` / `shell.scroll.up` controls in scope `shell.scroll`
   - inspect the safe `state` map (`y`, `max_y`, `viewport_height`, `can_scroll_down`).
3. Scroll semantically, not by coordinates:
   - preferred: `click_button(id="shell.scroll.down")`
   - alternate: `scroll(target="shell", direction="down", amount="page")` when the live app exposes the same observer path.
4. Verify movement before screenshot:
   - re-read `get_buttons(...)`
   - confirm `state.y` increased and `can_scroll_up` changed as expected.
5. Capture visual proof with `open_app_tab(... screenshot=true, force_relaunch=false, reap_stale=false)` or the primitive screenshot tool as appropriate.
   - Preserve the current running session; do not relaunch after scrolling unless explicitly trying to reset/rebuild.
   - If PrintWindow blanks after semantic gates pass, accept the Marionette internal Flutter screenshot fallback and label it honestly.
6. For Tony-facing delivery, send the PNG immediately with `MEDIA:<path>` plus only a compact note about scroll verification if useful.

## Fresh-build pitfall

If `shell.scroll` controls are absent in a fresh-looking session, the running Launcher may still be an older binary or built against the wrong target.

Recovery:
1. Kill/close only the running Launcher process if it locks `WebView2Loader.dll`.
2. Rebuild the QA target, not the regular target:
   - `flutter build windows --debug --target lib/main_marionette.dart`
3. Relaunch through Stage C MCP and re-check `get_buttons` for `shell.scroll.*` controls.

Stage C correctly rejects a regular `flutter build windows --debug` binary with `launch_wrong_debug_target_missing_marionette`; treat that as a freshness/build-target guardrail, not a product bug.

## Interpretation

A visible `Live Agent Terminal` header with no events is still valid visual proof of the terminal area, but it does not prove populated logs. Report the true state, e.g. `No redaction-safe task flow events have been recorded yet.` The product contract remains a redaction-safe structured operator transcript, not raw hidden chain-of-thought or unsafe logs.
