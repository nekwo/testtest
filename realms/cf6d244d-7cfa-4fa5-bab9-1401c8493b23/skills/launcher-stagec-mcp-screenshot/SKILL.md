---
name: launcher-stagec-mcp-screenshot
description: "Eternia Launcher Stage C MCP-first screenshot workflow: launch/open via MCP, login via MCP if needed, navigate via MCP, then capture current window with mcp_launcher_qa_screenshot_window and send MEDIA."
version: 1.0.0
metadata:
  hermes:
    tags: [eternia-launcher, stage-c, mcp, screenshot, windows, qa]
    related_skills: [native-mcp, flutter-ui-development, obsidian]
---

# Launcher Stage C MCP Screenshot Workflow

## Linked references

- `references/posts-algorithm-controls-visual-proof.md` — Posts right-rail Algorithm Controls proof lesson: reject stale/empty/loading/overflow screenshots, rebuild stale binaries, degrade read-only config hydration to fallback on backend HTML/500, and require visible sliders/no Flutter overflow before attaching screenshot proof.
- `references/library-mcp-semantic-action-pitfall.md` — Library semantic MCP item clicks can select rather than open; enumerate controls, use explicit details action, verify `library.details`, and respect scroll target schema.
- `references/mission-control-screenshot-qa-lane.md` — Mission Control / Agent Runtime Harness UI changes need live Stage C PNG proof when available; if skipped or blocked, explicitly report `live Stage C screenshot QA not performed` and the reason.
- `agent-runtime-harness` skill, `references/mission-control-active-surface-cleanup-and-neko-copy.md` — related Mission Control cockpit lesson: healthy Harness state can still show stale Ready for Archive cards or legacy PM copy; treat those as Launcher/read-model/UI-copy gaps, not runtime failure.
- `references/mission-control-exact-panel-screenshot-fallback.md` — if MCP can launch/capture the shell but the exact Mission Control panel is not semantically exposed, capture nearest supported shell evidence, rely on deterministic exact-panel tests/build, and explicitly report the exact-panel screenshot limitation without coordinate-clicking.
- `references/stagec-blank-visual-after-semantic-nav.md` — when MCP semantic state says the app is mounted/navigated but screenshots are blank, distinguish capture-method flake from true rendered-visual blocker and report visual QA as failed until pixels prove otherwise.
- `references/stagec-smoke-kubeconfig-credential-preflight.md` — Stage C smoke credential preflight for `auth_secret_unavailable`: verify Windows kubeconfig, Rancher auth/TLS, secret visibility, and the pragmatic `insecure-skip-tls-verify` recovery without printing secrets.
- `references/stagec-rancher-kubeconfig-tls-reset.md` — Stage C screenshot/browser-login credential failures caused by Tony's Windows Rancher kubeconfig reset/TLS CA mismatch; safe checks and pragmatic `insecure-skip-tls-verify` recovery.
- `references/stagec-smoke-credential-source-vs-runner-access.md` — distinguish provisioned Stage C agent credentials from the active runner being unable to reach k8s/Credential Manager sources; report the latter as credential-access blocked, not “credentials do not exist.”
- `references/mission-control-direct-tab-and-delivery.md` — Mission Control screenshot delivery lesson: if the MCP tab enum wrapper is stale, use the redaction-safe QA control REST `/qa/setTab` path when port/nonce are already available, capture with `screenshot_window`, deliver the PNG immediately, and do not let optional golden validation block Tony's screenshot request.
- `references/mission-control-video-proof-and-snapshot-parity.md` — Mission Control video proof lesson: recording a Harness goal is not UI proof unless the Launcher visibly shows matching active/blocked mission cards/counts; verify CLI↔Launcher parity before and after recording, and classify mismatches as live bridge/read-model bugs.
- `references/mission-control-video-recording-and-live-goal-smoke.md` — Mission Control video proof pattern: bounded FFmpeg `gdigrab` recording, run a tiny Harness goal during capture, verify MP4 finalization with `ffprobe`, and report the final Harness state/blocker.
- `references/mission-control-window-title-video-capture.md` — Mission Control video capture fallback: use FFmpeg `gdigrab` with the exact Launcher window title and sample a frame so desktop capture does not accidentally record the browser/foreground app.
- `references/fullscreen-screenshot-and-video-evidence-policy.md` — Tony's visual proof policy: maximize/fullscreen Launcher before screenshots/recordings; use PNG/key-frame evidence for model analysis; keep videos as human proof unless motion/timing is the defect.
- `references/mission-control-launcher-qa-smoke-env-parity.md` — Launcher QA Smoke env pinning/parity lesson: pin the Launcher child process to the intended Harness runtime root, distinguish real-empty vs bridge-unavailable vs parity-mismatch, and require CLI↔Launcher parity before accepting Mission Control visual proof.
- `references/mission-control-smoke-pinning-and-empty-goals.md` — Mission Control smoke lesson: an empty Active Missions UI is only interpretable after the launch envelope proves `hermes_profile`, `harness_runtime_root`, and `hermes_home` pins; if normal env shows goals while pinned smoke does not, identify the alternate Harness store/profile before declaring fixed/broken.
- `references/mission-control-current-video-proof-and-env-pinning.md` — current-proof lesson: if a screenshot shows the fixed state but a sent video does not, treat the MP4 as stale, re-record the exact fullscreen Launcher window, sample/verify a key frame, and require env-pinned Stage C launch metadata before sending.
- `references/mission-control-run-inspector-playback-proof.md` — run-inspector/playback proof lesson: after rebuild, capture exact Mission Control PNG, but distinguish static playback/archive proof from populated Run Inspector proof when the live Harness store has zero active runs.
- `references/mission-control-harness-attachment-contract-and-ui-proof.md` — attachment-capable Harness messaging plus Mission Control terminal/UI test lessons: structured image/video/file handles, capability allowlist, semantic agent-row selectors, bounded drawer pumps, first-event expansion, and honest non-AAA visual proof interpretation.
- `agent-runtime-harness` skill, `references/mission-control-archive-ux-and-terminal-contract.md` — archive/history UX contract: Ready for Archive + Mission Playback + Archive Controls belong in one unified panel; View Archive must toggle visible state; Live Agent Terminal is a redaction-safe structured operator transcript, not raw hidden CoT/log everything.
- `agent-runtime-harness` skill, `references/mission-control-linked-archive-read-model.md` — archive data-link contract: if Mission Control shows `No archived missions yet`, verify Harness `deleted_archive/` and `hermes harness snapshot --json` `archived_tasks`; visual empty-state proof is invalid until the archive source is known to be linked or truly empty.
- `references/mission-control-terminal-screenshot-white-render.md` — terminal screenshot lesson: semantic Mission Control state can be healthy while Windows pixels are a white render surface; do not send blank proof, classify the visual blocker, and treat block-glyph golden fallback as layout-only unless readable fonts are fixed.
- `references/mission-control-semantic-shell-scroll-terminal-proof.md` — Mission Control terminal/right-side scene proof: verify `shell.scroll.*` semantic controls, scroll by semantic button, prove `state.y` changed, preserve the session for screenshot, and rebuild `lib/main_marionette.dart` if the running binary lacks the scroll observer.
- `references/mission-control-right-inspector-terminal-proof.md` — right-inspector proof lesson: generic shell scroll can reach page bottom while the Run Inspector terminal/proof controls remain below the right panel fold; require visible terminal/proof pixels or report a page-local inspector semantic-scroll gap instead of overclaiming.
- `references/stagec-marionette-internal-screenshot-fallback.md` — when Stage C semantic gates pass but Win32/PrintWindow returns blank pixels, call the live VM-service `ext.flutter.marionette.takeScreenshots` direct extension path, decode the base64 PNG, and label it as internal Flutter screenshot fallback proof.
- `references/stagec-shell-scroll-and-mission-control-archive-proof.md` — use generic `shell.scroll` semantic controls plus scroll-position telemetry to reveal below-fold Mission Control archive/history proof; do not coordinate-scroll.
- `references/mission-control-spatial-semantic-controls.md` — Mission Control spatial/game and right-inspector surfaces need stable semantic controls for agent selection, terminal/inspector scroll, stage jumps, and proof inspection; coordinate clicks/resize-only screenshots are debug fallbacks, not AAA proof.
- `references/mission-control-drawer-semantic-controls-and-agent-console.md` — Mission Control drawer semantic-control contract: use `mission_control.drawer.*` IDs, verify selected state after click, capture the Agent Console drawer, and handle production DM stack layout/selection pitfalls.
- `references/mission-control-agent-chat-mcp-controls.md` — Mission Control Agent Console proof contract: expose selected Operator Channel action controls under `mission_control.agent_chat`, update app/bus/MCP schema allowlists together, account for stale loaded tool schemas, and treat “MCP click queued” as a product gap until a response/running state/blocker appears.
- `references/mission-control-persona-console-live-smoke.md` — live smoke pattern for Tony asking if he can talk to Neko/personas: rebuild Marionette if needed, open Agent Console, verify persona selectors plus `mission_control.agent_chat` controls, click `send_test_message`, and state the boundary versus full freeform message/response proof.
- `references/snipping-tool-bmp-conversion.md` — Snipping Tool BMP screenshots from Launcher/chat proof should be converted to PNG for delivery and vision/QA; use JPEG only for photo-like/lossy cases Tony explicitly wants.



## Core rule

Use the **MCP control path end-to-end**. For Tony-facing visual proof, make the Launcher window foreground and fullscreen/maximized before capturing screenshots or starting a recording. Use PNG screenshots or sampled key frames for model/vision analysis by default; keep MP4/video artifacts as human proof unless the bug depends on motion/timing/transitions.

When Tony asks for a specific Mission Control lower panel such as the **Live Agent Terminal**, scroll down to that exact target first and verify the PNG visibly contains the requested terminal/proof/stage area before sending. Do not substitute an overview, a Run Inspector header, or a resize-only workaround as exact-panel proof. If the requested panel is inside the spatial/game/right-inspector surface and semantic controls are missing, record the semantic-control gap and treat coordinate clicks/resizing as debug fallback only. See `references/mission-control-spatial-semantic-controls.md`.

1. Launch/attach the Launcher with MCP.
2. Login/authenticate with MCP if needed.
3. Navigate to the requested tab/page with MCP.
4. Capture the current visible window state with the primitive MCP screenshot tool.
5. Send the PNG directly as `MEDIA:<absolute-path>` unless Tony asks for analysis.

For Mission Control / Agent Runtime Harness Launcher visibility, **reuse the existing Stage C smoke credential path** (`credential_profile: stagec-smoke`, `browser_login: true`). Do not invent a new credential system or ask Tony for credentials unless the existing Stage C smoke profile fails. Mission Control should display only redaction-safe auth state such as authenticated/login_required/failed.

For user-facing Mission Control UI changes, live Stage C screenshot QA is the standard visual proof lane when available. It must not block deterministic non-UI Harness/backend implementation runs, but if it is skipped or blocked the report must explicitly state `live Stage C screenshot QA not performed` with the blocker reason; see `references/mission-control-screenshot-qa-lane.md`. Do not count “rebuilt/relaunched Stage C” as screenshot QA by itself: the proof lane completes only after a fresh PNG path is captured/reported or the exact blocker is named. If the exact Mission Control panel is not exposed by the current MCP tab enum/button registry, do not coordinate-click; capture the nearest supported shell screenshot as partial evidence, pair it with exact-panel widget/build proof, and explicitly report the exact-panel screenshot limitation; see `references/mission-control-exact-panel-screenshot-fallback.md`. If the in-chat MCP wrapper schema rejects `missionControl` but a live QA marionette session already exposes the redaction-safe port/nonce, the bounded direct QA REST `/qa/setTab` route is an acceptable fallback for navigation; still use `mcp_launcher_qa_screenshot_window` for the actual visual proof and record the wrapper/schema mismatch.

When Tony asks for a screenshot, the deliverable is the captured PNG. Do not block the response on optional golden-test validation or visual-regression artifact generation after a live screenshot is already captured. If extra validation fails because of test-only asset/font setup, clean up temporary test files, report the validation blocker separately, and send the PNG immediately.

For Mission Control video/screenshot proof, the artifact must visibly show the exact claimed UI state. If a live Harness goal is created or run, verify the Launcher pixels show the corresponding Active Missions card/counts. CLI/Harness success without visible Active Missions evidence is not valid UI proof; report the Launcher/read-model mismatch as a high-severity regression instead of presenting the artifact as success.

When a card/user provides a screenshot PNG path as evidence, treat the path itself as the evidence handle. Do **not** read or base64-load PNG bytes unless visual inspection is explicitly required; record the path, source tool, dimensions/byte_count if already available from the capture envelope, and proceed.

Do **not** start by filesystem-searching helper scripts or hand-driving unrelated PS1/direct-control flows. Helpers are allowed only as bounded MCP invocation wrappers when first-class in-chat MCP tools are not exposed/stale; they still call the MCP server tools.

## Current validated command sequence

From repo root `X:\Unreal Engine\Engine\Launcher\EterniaLauncher`, using the bounded MCP helper:

```bash
cd 'X:/Unreal Engine/Engine/Launcher/EterniaLauncher'

powershell.exe -NoProfile -ExecutionPolicy Bypass \
  -File docs/stages/qa-reboot/scripts/Invoke-LauncherQaMcpTool.ps1 \
  -Tool mcp_launcher_qa_open_app_tab \
  -ArgsJson '{"tab":"library","browser_login":true,"credential_profile":"stagec-smoke","screenshot":false,"reap_stale":true}' \
  -CallTimeoutSeconds 240

powershell.exe -NoProfile -ExecutionPolicy Bypass \
  -File docs/stages/qa-reboot/scripts/Invoke-LauncherQaMcpTool.ps1 \
  -Tool mcp_launcher_qa_screenshot_window \
  -ArgsJson '{"window_title_prefix":"Eternia Launcher","label":"alice_library_current","out_dir":"X:/tmp/stagec/screenshots"}' \
  -CallTimeoutSeconds 120
```

Validated live proof on 2026-05-18:

- `mcp_launcher_qa_open_app_tab(library, browser_login=true, credential_profile=stagec-smoke, screenshot=false, reap_stale=true)` returned `ok=true`, `auth_state.status=authenticated`, `navigation_state.selected_tab=library`, `window_visible=true`, `qa_control_ready=true`.
- `mcp_launcher_qa_screenshot_window(window_title_prefix='Eternia Launcher', label='alice_library_current', out_dir='X:/tmp/stagec/screenshots')` returned `ok=true`, `capture_method=printwindow`, `image_path=X:\tmp\stagec\screenshots\alice_library_current_20260518103646064.png`, `byte_count=1076666`, dimensions `1266x793`, redaction safe.

## Tool semantics

### `mcp_launcher_qa_open_app_tab`

Use for launch/attach + auth/navigation. Important args:

- `tab`: requested surface, e.g. `library`, `news`, `posts`, `shop`, `downloads`, `ai`.
- `browser_login: true`: allow MCP to perform the browser-login fallback if needed.
- `credential_profile: stagec-smoke`: use the Stage C smoke credentials/profile.
- `screenshot: false` when you intend to call the primitive screenshot tool afterward.
- `reap_stale: true` for clean acceptance screenshots unless intentionally reusing a known healthy session.

Acceptance fields before screenshot:

- `ok=true`
- `app.attached=true`
- `app.window_visible=true`
- `app.qa_control_ready=true`
- `auth_state.status=authenticated`
- `navigation_state.selected_tab=<requested tab>`
- `navigation_state.blocking_modal_present=false`

### `mcp_launcher_qa_screenshot_window`

This is a primitive, not an orchestrator. It screenshots whatever the current window is showing.

Use after navigation/click/login is already done.

Typical args:

```json
{
  "window_title_prefix": "Eternia Launcher",
  "label": "alice_library_current",
  "out_dir": "X:/tmp/stagec/screenshots"
}
```

Acceptance fields:

- `ok=true`
- stable top-level `image_path`
- `capture_method` / `capture_method_used` = `printwindow`
- non-trivial `byte_count`
- `bounds.width` / `bounds.height`
- `redaction.safe=true`

If `ok=false` with `helper_window_not_found`, the app/window was not visible or not launched; go back to MCP `open_app_tab`/launch, do not debug screenshots first.

## User-facing delivery

When Tony asks “send me it”, respond with the native media attachment and minimal text. If he asks for a specific lower panel or “the terminal dingus,” first scroll to that exact panel until it is visibly in frame; do **not** substitute an overview, a top-of-inspector shot, or a resized workaround unless scrolling cannot expose the target and you explicitly report that blocker.

If Tony provides Snipping Tool screenshots as BMP files, convert them to PNG before delivery/analysis by default. PNG is the right proof format for UI/text screenshots; do not JPEG-compress Launcher/Mission Control/code/terminal screenshots unless Tony explicitly asks for lossy smaller files. See `references/snipping-tool-bmp-conversion.md`.

```text
MEDIA:X:\\\\tmp\\\\stagec\\\\screenshots\\\\alice_library_current_20260518103646064.png
```

For video requests, maximize/fullscreen the Launcher first, record with a bounded duration so the MP4 finalizes cleanly, verify with `ffprobe`, then deliver the MP4 the same way. For Mission Control video proof, prefer recording the exact Launcher window title (`gdigrab -i title='Eternia Launcher (stagec-smoke)'`) rather than `-i desktop` after a short sample-frame sanity check; desktop capture can record a browser or other foreground app even when the Launcher is running. If Tony says the current screenshot is correct but the video is not, do not defend or resend the old MP4: treat it as stale proof, re-record the current fullscreen Launcher window, sample a key frame, verify the key frame shows the claimed state, then send the fresh MP4. See `references/mission-control-window-title-video-capture.md` and `references/mission-control-current-video-proof-and-env-pinning.md`.

```text
MEDIA:C:\\\\Users\\\\beast\\\\AppData\\\\Local\\\\EterniaLauncher\\\\stagec-smoke-local\\\\videos\\\\mission_control_test_goal_<timestamp>.mp4
```

Treat video as **human proof by default**. Do not send full videos to model vision unless motion/timing is the actual bug; if the question is static and a video already exists, sample one key frame or capture a PNG instead. Do not add visual analysis unless requested. If a Harness smoke goal ran during the capture, include only the compact final state/blocker unless Tony asks for logs.

## Video proof preflight

Before starting FFmpeg for Mission Control proof, verify the Launcher is already on the intended page and the QA control path is ready. If an env-pinned Stage C direct launch reports `window_visible=true` but `qa_control_ready=false`, do **not** start recording yet and do **not** present a partial/invalid video attempt. First diagnose the QA-control bind/readiness path or relaunch through the bounded MCP/open-tab workflow, then record only after navigation/parity can be verified.

## Pitfalls

- When Tony says the target is “the terminal,” “the dingus,” or another specific lower panel, the required proof is that exact panel after scrolling down to it. Do not send the page overview, top Run Inspector metadata, or a window-resize workaround as the answer. Use semantic scroll controls first; only if the target still cannot be exposed should you report the exact QA-control/layout gap.
- For Mission Control video proof, prefer a bounded FFmpeg recording (`ffmpeg -f gdigrab ... -t <seconds> ...mp4`) over a background recording that must be killed. Force-killing FFmpeg can leave an invalid MP4 with `moov atom not found`; verify the final artifact with `ffprobe` before sending. When recording Mission Control specifically, prefer exact window-title capture (`-i title='Eternia Launcher (stagec-smoke)'`) plus one sampled-frame sanity check; desktop capture can accidentally record a browser/foreground app. See `references/mission-control-video-recording-and-live-goal-smoke.md` and `references/mission-control-window-title-video-capture.md`.
- For Launcher QA Smoke Mission Control proof, pin the Launcher child process to the same Harness runtime root/profile used by CLI checks before trusting empty/active mission state. **Always pass explicit `hermes_profile`, `harness_runtime_root`, and `hermes_home` on `open_app_tab` / `launch_or_attach` smoke runs**, then verify the launch envelope reports `app.hermes_profile` non-null, `app.harness_runtime_root_configured:true`, and `app.hermes_home_configured:true`. If the envelope is unpinned (`hermes_profile:null` or `harness_runtime_root_configured:false`), treat the screenshot/video as invalid smoke proof even if navigation/auth succeeded; rerun pinned instead of diagnosing UI state. Distinguish real-empty (`open_tasks == 0` and mapped active goals `0` from the exact pinned root) from bridge-unavailable and parity-mismatch states. If Tony's normal Launcher shows goals but pinned smoke shows none, first compare the exact Harness store/root/profile for normal vs smoke; this is usually a root/profile mismatch, not proof that Mission Control mapping regressed. If CLI reports open/blocked/running work while Launcher maps zero active goals for the same pinned root, render/report a bridge diagnostic intervention rather than accepting empty UI as proof. If the helper script supports env-pinning flags but the launch envelope still shows unpinned fields after passing pins, audit the MCP launch manager/composer (`buildLaunchArgs`, open-tab launch composition) for missing argument forwarding before changing Mission Control UI mapping. Post-fix, preserve the guardrail that pinned requests refuse unpinned live disk sessions with a bounded env-pin mismatch instead of silently attaching. See `references/mission-control-launcher-qa-smoke-env-parity.md`.
- Schema first: before calling page-local tools such as `scroll`, check the MCP tool's supported targets and current semantic buttons. Do not call `scroll` for Library unless the schema explicitly exposes a Library scroll target; current scroll targets may be limited to `shell`, `news.feed`, and `posts.feed`. For Library, treat item clicks as select/focus actions and use a separate explicit open/details action, then verify `library.details`; see `references/stagec-mcp-scroll-click-sequencing.md` and `references/library-semantic-click-scroll-pitfalls.md`.
- If `browser_login_helper_failed` / `auth_secret_unavailable` appears, distinguish **credential contract existence** from **active runner access**. The Stage C agent credential path may already be provisioned (`stagec-smoke`, `qa-stagec-smoke`, k8s `eternia-staging/stagec-smoke-credentials`, Windows Credential Manager `EterniaStageC/stagec-smoke`) while the current runner cannot reach it because kube auth is expired, Credential Manager fallback is absent, or interactive fallback is noninteractive. Report: “credential path exists/provisioned, current runner cannot access a source,” not “credentials do not exist.” See `references/stagec-smoke-credential-source-vs-runner-access.md`.
- The bounded PowerShell helper is still an MCP invocation path. Do not call it “not MCP”; it launches/calls the Stage C MCP server with safe timeouts and envelopes.
- First-class in-chat MCP tools may be stale if loaded before MCP server rebuild/reload. If a fresh helper sees the new tool but the chat namespace does not, report a host/session reload need and continue with the bounded MCP helper.
- Rebuild the canonical `tool/stagec_qa_mcp_server/build/stagec_qa_mcp_server.exe` after source/tool catalog changes before expecting Hermes MCP discovery to list new tools.
- Do not use the old script-first/direct-control News recipe as the primary workflow. It is fallback/historical debugging only.
- Do not accept `click_button` alone as visual proof for page-local controls; verify state with MCP state/buttons and then screenshot current window. If semantic state and screenshot disagree, the screenshot/visible UI is authoritative for user-facing claims and the mismatch is a QA bug, not a PASS.
- For Mission Control screenshots or videos, if `mcp_launcher_qa_set_tab` or `open_app_tab` cannot navigate because the wrapper schema rejects `missionControl`, do not fall back to coordinate clicks. If a live marionette control port/nonce are already available from the current launch artifact, call the QA control REST `/qa/setTab` endpoint with the nonce, verify navigation state, then capture via `mcp_launcher_qa_screenshot_window`. Report the schema mismatch as a tool wrapper gap. For video proof, additionally verify the Launcher UI itself shows the expected active/blocked mission cards or counts from `hermes harness snapshot --json`; a recording of a Harness run while the UI stays empty is not Mission Control proof, it is a high-severity live bridge/read-model parity bug. See `references/mission-control-direct-tab-and-delivery.md` and `references/mission-control-video-proof-and-snapshot-parity.md`.
- For requested screenshot delivery, optional golden tests are not part of the user-facing proof contract unless Tony explicitly asks for visual-regression testing. A live PNG path from `screenshot_window` is enough to answer “show me how it looks.” If a temporary golden test hits `google_fonts`/asset loading trouble, stop retrying unchanged, delete the temp artifact, and deliver the screenshot with the blocker noted separately.
- If Stage C Mission Control proof uses the direct/in-chat typed wrapper and the enum or semantic button registry does not expose `missionControl`, try the bounded MCP helper/open-tab path before falling back to weaker evidence. A freshly rebuilt helper may accept `missionControl` even when the active chat wrapper schema or button registry is stale; record the schema/registry freshness gap rather than coordinate-clicking.
- For Mission Control Agent Console/chat proof, screenshots and drawer toggles are not enough. The selected Operator Channel must expose stable semantic controls under `mission_control.agent_chat` (for example `mission_control.agent_chat.send_test_message` and `mission_control.agent_chat.action.run_one_turn`) so QA can click real chat/actions by id. If Tony asks whether he can talk to Neko or each persona, use the live persona-console smoke: rebuild the Marionette target if MCP reports `launch_wrong_debug_target_missing_marionette`, open `mission_control.drawer.agents`, verify persona selectors for Neko/Launcher Dev/Backend Dev/QA, verify `mission_control.agent_chat` controls, click `send_test_message`, and capture a PNG. Report the boundary honestly: selector + test-message action proof is not full freeform typed-message/response proof unless that transcript loop was also exercised. If the app emits the controls but `get_buttons()` reports `registry_dropped_reasons: unknown_scope`, update the app marionette hook, `StageCQaCommandBus` known scope/kind allowlists, and Stage C MCP server tool schema enum together. If the current chat’s loaded MCP schema still rejects the new scope, use unfiltered `get_buttons()` plus id-based `click_button` as a live workaround and report that a Hermes/MCP reload is needed for scoped queries. See `references/mission-control-agent-chat-mcp-controls.md` and `references/mission-control-persona-console-live-smoke.md`.
- If `PrintWindow` / `screenshot_window` still returns `helper_blank_capture` after semantic gates pass (`authenticated`, `selected_tab=missionControl`, QA control ready, no blocking modal), use the Marionette internal Flutter screenshot fallback before declaring screenshot proof impossible. Extract the VM service URI from the child stdout log, call `ext.flutter.marionette.takeScreenshots` as a direct VM-service extension path, decode the returned base64 PNG, sanity-check it, and label the artifact as internal Flutter screenshot fallback proof. Do not use the `callServiceExtension?...method=ext.flutter.marionette.takeScreenshots` shape unless tested; it can report `Method not found` even while the direct extension path works. See `references/stagec-marionette-internal-screenshot-fallback.md`.
- When a Mission Control screenshot shows `0 active missions` / `0 active runs`, do not overclaim populated run-inspector proof. It can prove static surfaces such as `Mission Playback`, `Archive Controls`, and active/archive separation, but a populated `Run Inspector` requires a live/seeded run or widget-test proof. Report this as partial visual proof unless the run inspector is actually visible with run rows.
- For Mission Control archive/history changes, visual proof must show the actual unified surface when claiming live UX proof: `Ready for Archive`, `Mission Playback`, and `Archive Controls` should be in one panel, and `View Archive` must visibly change state. If the unified section is below the fold, use the generic semantic shell scroll controls (`get_buttons(scope="shell.scroll")`, then `click_button(id="shell.scroll.down")`/`shell.scroll.bottom`) and verify scroll telemetry (`y`, `max_y`, `can_scroll_down`) changes before capturing proof. Do not coordinate-scroll/click or overclaim the screenshot. If `shell.scroll` is missing on a content-rich shell page, report a semantic-control regression/gap and pair nearest live PNG proof with deterministic widget tests. See `references/stagec-shell-scroll-and-mission-control-archive-proof.md` and `agent-runtime-harness` reference `mission-control-archive-ux-and-terminal-contract.md`.
- When Tony asks for a Mission Control terminal screenshot, the deliverable must be a readable PNG of the right-side `Run Inspector` / `Live Agent Terminal`, not merely semantic MCP state. If the terminal/right-side scene is below the fold, use `get_buttons` to verify `shell.scroll.*` controls, click semantic `shell.scroll.down`, verify `state.y` increased, then capture the same running session; do not coordinate-scroll or relaunch after scrolling unless rebuilding. If `shell.scroll` controls are absent, rebuild the QA target with `flutter build windows --debug --target lib/main_marionette.dart` and relaunch before retrying. If `shell.scroll.bottom` reaches `y=max_y` but the screenshot still only shows the top Run Inspector metadata and not `Live Agent Terminal`, `Terminal index`, transcript rows, `Inspect proof image`, or `Proof Image Inspector`, classify it as partial proof plus a page-local right-inspector semantic-scroll gap; do not overclaim exact terminal/proof pixels. If MCP reports `selected_tab=missionControl`, shell mounted, no loading overlay, and no error banner, but PrintWindow and foreground captures are uniform white, classify it as `live Windows pixel proof blocked: white render surface` and do **not** send the blank image as proof. A widget/golden fallback may be used only as clearly-labeled partial evidence; if GoogleFonts/test capture renders text as block glyphs, treat it as layout-only and not readable terminal proof. See `references/mission-control-terminal-screenshot-white-render.md`, `references/mission-control-semantic-shell-scroll-terminal-proof.md`, and `references/mission-control-right-inspector-terminal-proof.md`.
- If rebuilding the Windows debug executable fails because `WebView2Loader.dll` is locked by `eternia_launcher`, close/kill the Launcher and rebuild, then relaunch for screenshot proof; this is a freshness/lock recovery step, not a product defect.
- If Stage C authenticated screenshot QA fails with `auth_secret_unavailable`, do not conclude the credentials were never provisioned. First check Tony's Windows Rancher kubeconfig at `C:\Users\beast\.kube\config` / `/c/Users/beast/.kube/config`: `kubectl` may be present while the kubeconfig token is stale/reset or its embedded `certificate-authority-data` mismatches Rancher's public proxy cert. Use redaction-safe checks (`kubectl auth can-i ...`, `kubectl get secret ... -o name`, key names only) and, if the Rancher-generated config fails TLS but works with `--insecure-skip-tls-verify=true`, restore the pragmatic working setting with `kubectl config set-cluster local --insecure-skip-tls-verify=true`. See `references/stagec-rancher-kubeconfig-tls-reset.md`.
- If MCP reports authenticated/mounted/navigated state but `screenshot_window` returns repeated `blank_capture`, bring the Launcher foreground and take a bounded diagnostic foreground capture only to separate capture-method compatibility from true visual rendering failure. If the foreground capture is also blank, switch to a known-good tab once; if still blank, classify the result as a high-severity visual QA blocker instead of describing the target screen as visually reviewed. See `references/stagec-blank-visual-after-semantic-nav.md`.
- If browser login fails with `auth_secret_unavailable`, do **not** conclude the Stage C agent credentials were never created. First diagnose the credential source chain precisely: Windows `kubectl` should be reachable from Git Bash/MSYS, default kubeconfig is usually `C:\Users\beast\.kube\config`, and the helper expects `eternia-staging/stagec-smoke-credentials` with `username`/`password` keys. Safe checks are `kubectl config current-context`, `kubectl auth can-i get secret/stagec-smoke-credentials -n eternia-staging`, and `kubectl -n eternia-staging get secret stagec-smoke-credentials -o name`. If those fail with `Unauthorized`, classify stale kube token/session; if they fail with `x509: certificate signed by unknown authority` after config restore, verify/repair the kubeconfig CA or use the established Rancher-context workaround (`kubectl --insecure-skip-tls-verify=true ...` / cluster config set) before blocking screenshot QA. Never print secret `.data` values; only report secret name/key names.
- Do not overclaim interactive selection from an unverified helper/session path. Low-level interactive calls such as `get_buttons` and `click_button` must be proven against the currently rebuilt MCP server/profile. If they return `app_not_attached`, treat it as a session-boundary/freshness issue and fix/reload the MCP path rather than coordinate-clicking.
- Library item selection is expected when the live MCP schema exposes `library.item` and `library.navigation`, but item controls must be **verb-explicit selectors**: `library.item.select.<safe-id>`, `scope=library.item`, `kind=select`, label `Select <title>`. Verify with `get_buttons(scope=library.item)`, semantic `click_button`, selected-state readback, and screenshot proof.
- Do not treat a Library item selector as an open/details action. Correct flow is: click `library.item.select.<safe-id>` to select/focus, then click `library.action.open_details` to open the selected item's details/post, then verify `get_widget_state(widget=library.details)` has `details_open=true` and `selected_matches_details=true`.
- Library item selection is now expected when the live MCP schema exposes `library.item` and `library.navigation`. Verify with `get_buttons(scope=library.item)`, semantic `click_button`, selected-state readback, and screenshot proof. If a stale schema exposes only `shell.nav`, `blocking_modal`, and `library.view_mode`, do not claim item selection until the MCP server EXE/profile is rebuilt/reloaded.
- Library item controls must be verb-explicit. Treat `library.item.select.<safe-id>` / label `Select <title>` / kind `select` as the selector-only surface, and use `library.action.open_details` as the separate open action. If an item control is named like `library.item.<safe-id>` or kind `item`, that is ambiguous and should be fixed rather than normalized in agent workflow.
- When a page visibly has local tabs/subtabs but `get_buttons()` only exposes `shell.nav`, record a semantic-control gap instead of coordinate-clicking. For Education this gap is now closed: `get_buttons()` should expose `education.nav.{library,explore,studio,tutor,passport,enterprise,demand,power_user}` when the Education root tabs are mounted, and agents should click those IDs directly (for example `education.nav.explore`) then verify the selected tab via `get_buttons()` (`selected:true`, `enabled:false` on the active subtab). If Education subtabs disappear from the semantic surface again, treat it as a regression.
- Education Explore OCW controls are learner/product actions, not Studio authoring shortcuts. If `ADD TO ETERNIA` copies an OCW URL or tells the user to open Studio / paste into “Import from MIT OCW,” classify it as a product bug: the intended flow is one-click learner add/install/enroll as-is through the server pipeline, followed by Library/Education Library visibility. If the backend endpoint is missing, surface a truthful unavailable/pending state and create a backend card; do not normalize copy/paste-to-Studio as success.
- If a new Education Explore rail looks like a carousel, verify the carousel actually moves. Prefer reusing existing Launcher carousel/controller logic; only modify shared logic carefully, and duplicate as a last resort with a documented reason. A static rail dressed as a carousel is not acceptable QA pass.

See `references/stagec-mcp-library-item-selection-gap.md` for the original gap evidence, `references/stagec-library-semantic-controls.md` for the `library.item` / `library.navigation` control contract and smoke pattern, `references/stagec-library-visual-semantic-parity.md` for the visual-vs-semantic mismatch rule where screenshot evidence overrides selected-state readback, `references/stagec-semantic-control-naming-and-page-local-tabs.md` for the 2026-05-18 live lesson on verb-explicit IDs and Education subtab gaps, `references/stagec-mcp-scroll-click-sequencing.md` for schema-first scroll/click sequencing and Library select-then-open verification, and `references/stagec-mcp-kanban-efficiency.md` for lean Kanban execution patterns around Stage C MCP cards.
