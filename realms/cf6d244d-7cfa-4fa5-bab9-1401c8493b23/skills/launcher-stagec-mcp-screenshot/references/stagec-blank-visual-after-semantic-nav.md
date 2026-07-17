# Stage C blank visual after semantic navigation

## Trigger

Use this reference when Stage C MCP reports a healthy app/session/navigation state, but screenshot proof is blank or nearly uniform.

Observed pattern from a Mission Control/AI screenshot pass:

- `open_app_tab` succeeded after rebuilding the debug EXE with `lib/main_marionette.dart`.
- MCP state reported authenticated session, mounted shell, selected tab, wide window metrics, and visible semantic nav controls.
- `screenshot_window`/PrintWindow failed with `blank_capture` and uniform pixels.
- A foreground `CopyFromScreen` diagnostic capture also produced a blank white content area with only the window border.
- Switching tabs via `setTab` changed semantic state, but visual capture stayed blank.

## Correct classification

Do **not** accept semantic MCP state as visual proof when the pixels are blank. Treat semantic-vs-visual disagreement as a high-severity visual QA blocker until a non-blank screenshot proves the rendered UI.

This is different from a simple screenshot helper flake:

- If PrintWindow is blank but foreground `CopyFromScreen` shows the UI, the issue is capture-method compatibility.
- If both PrintWindow and foreground `CopyFromScreen` are blank while MCP state is mounted, the issue is likely app rendering/bootstrap/window-surface state and must block visual acceptance.

## Diagnostic sequence

1. If launch fails with `launch_wrong_debug_target_missing_marionette`, rebuild:

   ```powershell
   Set-Location 'X:\Unreal Engine\Engine\Launcher\EterniaLauncher'
   flutter build windows --debug --target lib/main_marionette.dart
   ```

2. Relaunch/navigate with MCP and verify state:

   - runtime entrypoint is `lib/main_marionette.dart`
   - `auth_state.status=authenticated`
   - `navigation_state.selected_tab=<target>`
   - `blocking_modal_present=false`
   - shell widget mounted
   - expected semantic nav controls visible

3. Try MCP `screenshot_window` first.

4. If `blank_capture` repeats, bring the Launcher foreground and take a one-off `CopyFromScreen` diagnostic capture. This is diagnostic fallback, not a replacement for MCP semantic navigation.

5. If the fallback is still blank, switch to a known-good tab such as Home and capture again. If that is also blank, classify it as a rendered-visual blocker rather than an AI/Mission-Control-only design issue.

## Reporting language

Report the exact distinction:

- "MCP semantic state is healthy, but visual screenshot proof is blank. This is a FAIL/intervention, not an accepted Mission Control screenshot."

Avoid saying the screenshot "looks good" or that Mission Control visual QA passed when only semantic state passed.
