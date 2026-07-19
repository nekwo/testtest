# Mission Control run-inspector / playback proof lesson

When implementing Mission Control UI changes that add playback/archive/run-inspector surfaces, visual proof must distinguish which surface is actually populated by the live Harness snapshot.

## Durable pattern

1. Rebuild the Marionette/Stage C debug executable before screenshot proof.
2. If a running `eternia_launcher.exe` locks `WebView2Loader.dll`, kill/close the Launcher and rebuild; this is a normal freshness recovery, not a product failure.
3. Navigate to Mission Control using the MCP/open-tab helper when the in-chat wrapper enum or semantic registry is stale. The helper can accept `missionControl` even if a direct typed wrapper lacks that tab.
4. Maximize/foreground the window before capture.
5. Capture a PNG with `screenshot_window` and inspect whether the exact claimed surfaces are visible.
6. If the Harness store has `0 active missions` / `0 active runs`, the screenshot can prove static/playback/archive surfaces but cannot prove a populated Run Inspector. Pair it with widget tests or seed/run a live mission before claiming populated run-inspector visual proof.
7. Report exact limitation language: “live screenshot proves playback/archive surface; run-inspector populated state covered by widget tests / requires a live Harness run for pixel proof.”

## Acceptance distinction

- PASS: screenshot visibly shows `Mission Playback`, `Archive Controls`, active/archive separation copy, and/or a populated `Run Inspector` depending on available live state.
- PARTIAL: screenshot shows Mission Control and new playback/archive sections, but no active run exists, so the populated Run Inspector is not visible.
- FAIL/BLOCKED: CLI/Harness says active runs exist but Launcher shows empty Mission Control; classify as a live bridge/read-model parity bug, not successful UI proof.
