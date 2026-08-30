# Stage C Marionette Internal Screenshot Fallback

## Trigger

Use this when Stage C semantic gates pass but Win32/PrintWindow capture returns blank or low-information pixels.

Typical evidence:

- `open_app_tab`/navigation reports `ok=true`, authenticated, `selected_tab=<target>`, `blocking_modal_present=false`, QA control ready.
- Screenshot helper fails with `helper_blank_capture` or `helper_low_information_capture`.
- Helper diagnostics show `blank_pixel_ratio` near `1` and `unique_color_count` near `1`.
- Foreground/PrintWindow capture may be white/blank even when the Flutter tree is actually rendered.

## Correct response

Do **not** send the blank PNG as proof. Do **not** treat semantic state alone as visual proof. First try the internal Flutter-render screenshot fallback.

## Production MCP behavior to preserve

`mcp_launcher_qa_open_app_tab(..., screenshot:true)` should, after semantic gates pass:

1. Invoke the normal PrintWindow screenshot helper.
2. If helper returns blank/low-info for a content-rich tab, parse the local child stdout log for the Dart VM service URI.
3. Call the VM-service extension as a **direct extension GET path**:
   - `ext.flutter.marionette.takeScreenshots?isolateId=<main-isolate-id>`
4. Decode the first returned base64 PNG.
5. Save it as `<scenario_label>_marionette_internal.png` under the screenshot/artifact dir.
6. Return `ok=true` with screenshot metadata:
   - `method: MarionetteInternalFlutter`
   - `fallback_from: PrintWindow`
   - `fallback_reason: helper_blank_capture` or `helper_low_information_capture`
   - `acceptance_mode: marionette_internal_fallback`
   - original helper diagnostics preserved.

If the fallback is unavailable or returns invalid/blank data, preserve the hard failure as `visual_render_mismatch` with both the helper diagnostics and fallback failure class.

## Important implementation details

- The VM service URI contains a local auth token. Never put it in MCP envelopes, chat output, trace-safe logs, or user-facing summaries.
- `open_app_tab` must launch with child log capture enabled; the VM-service URI is printed only in child stdout.
- On this Windows/Flutter VM-service build, `callServiceExtension?method=ext.flutter.marionette.takeScreenshots` can report `Method not found` even while the extension is registered and callable. Use the direct extension path unless a future verified implementation proves otherwise.
- Parse `getVM` first and use the main isolate id from the VM result.
- Sanity-check the PNG signature and dimensions before accepting it as proof.
- The fallback is acceptable visual proof because it captures Flutter's rendered scene, but label it clearly as internal Flutter screenshot fallback, not Win32 PrintWindow proof.

## Verification pattern

Minimum gates:

```bash
cd 'X:/Unreal Engine/Engine/Launcher/EterniaLauncher/tool/stagec_qa_mcp_server'
dart analyze lib/server.dart test/screenshot_envelope_test.dart
dart test test/screenshot_envelope_test.dart test/screenshot_args_test.dart test/sparse_tab_acceptance_test.dart test/open_app_tab_test.dart --reporter=compact
dart test --reporter=compact
```

Then rebuild the MCP server executable:

```bash
dart compile exe bin/main.dart -o build/stagec_qa_mcp_server.exe
```

If the EXE is locked, identify and kill only the exact `stagec_qa_mcp_server.exe` process, then recompile.

Live proof through the bounded MCP helper should return an envelope like:

- `ok: true`
- `screenshot.method: MarionetteInternalFlutter`
- `screenshot.fallback_from: PrintWindow`
- `screenshot.fallback_reason: helper_blank_capture`
- non-trivial `byte_count`, `width`, and `height`
- warning noting PrintWindow failed but internal fallback was accepted.

## Wrapper freshness pitfall

After killing/rebuilding the MCP server EXE, the in-chat MCP wrapper can be stale or circuit-broken. Do not keep retrying it blindly. Use the bounded helper invocation against the rebuilt EXE, then report that wrapper/session reload may be needed.
