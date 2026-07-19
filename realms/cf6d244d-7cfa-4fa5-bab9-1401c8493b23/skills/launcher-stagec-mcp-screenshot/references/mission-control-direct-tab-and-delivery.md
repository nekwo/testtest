# Mission Control direct-tab fallback and screenshot delivery

## When this applies

Use this reference for Eternia Launcher Mission Control visual proof when:

- the Launcher is already running under the Stage C QA marionette;
- the desired screen is Mission Control;
- first-class MCP screenshot tools can launch/capture the window, but the in-chat tab wrapper schema rejects `missionControl` or does not expose that page yet;
- Tony asked to see how the current Mission Control UI looks.

## Durable lesson

The user-facing proof for “show me how it looks” is the live PNG from `mcp_launcher_qa_screenshot_window`. Do not block screenshot delivery on optional golden-test generation once a live screenshot exists.

A golden test can be useful as follow-up visual-regression coverage, but it is not required to answer Tony’s screenshot request. If a temporary golden test fails due to test-only font/asset setup (for example `google_fonts` cannot load a font in Flutter tests), do not repeat the same command unchanged. Stop, clean up temporary test artifacts, deliver the PNG, and report the validation blocker separately if relevant.

## Safe fallback sequence

1. Launch/attach the Launcher with Stage C QA MCP as usual.
2. If MCP enum navigation does not accept `missionControl`, use the current marionette session’s redaction-safe QA control port/nonce files from the launch artifact and call the local QA REST endpoint:
   - `POST http://127.0.0.1:<port>/qa/setTab`
   - header: `X-Stagec-Qa-Nonce: <nonce>`
   - JSON body: `{ "tab": "missionControl" }`
3. Verify navigation state reports Mission Control selected.
4. Capture the actual visible window with `mcp_launcher_qa_screenshot_window`.
5. Send `MEDIA:<absolute_png_path>` to Tony with minimal commentary.
6. Record the MCP wrapper/schema mismatch as a tool gap; do not coordinate-click.

## Cleanup rule

If you created temporary screenshot/golden test files only to validate the capture and they failed or are no longer needed, remove them before final handoff so the product repo stays clean.

## Reporting pattern

Keep the user-facing response short:

```text
Got the actual Mission Control Launcher screenshot, Master.

MEDIA:<absolute_png_path>

Notes:
- PM is hidden from the scene.
- Active Missions / Agent Profiles reflect the current live state.
- Optional golden validation was skipped/blocked by <reason> if relevant.
```
