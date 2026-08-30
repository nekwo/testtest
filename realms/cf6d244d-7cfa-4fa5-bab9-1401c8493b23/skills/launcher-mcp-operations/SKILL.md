---
name: launcher-mcp-operations
description: "General-purpose Eternia Launcher MCP operation skill: drive, verify, and capture the Launcher through the launcher_qa MCP surface — launch/attach and env pinning, semantic controls and navigation, forms/scroll/tabs, library and board lanes, screenshots and video, credential preflight, and evidence delivery."
version: 2.0.0
metadata:
  hermes:
    surfaces: [mission_chat]
    modes: [standard]
    load_policy: recommended
---

# Launcher MCP Operations

How to operate the Eternia Launcher end-to-end through the `launcher_qa` MCP surface:
launch or attach it, navigate it, click its semantic controls, read its state back, and
capture screenshots and video as evidence. Screenshots are one lane in here, not the whole
skill.

> **Historical-vocabulary note (2026-07-30, rewritten 2026-08-28):** the harness
> goal/task mission lane was removed
> (`X:/Eternia/hermes-agent/docs/agent-runtime-harness/archive/2026-08-22-pre-consolidation/16-mission-lane-removal.md`).
> There are no goals, missions, runs, proof gates, task ids, `run-until-settled`, or
> Active Missions counts any more, and the projector/read-model is retired. Chat is the
> only lane. This package was rewritten on 2026-08-28 to strip mission-era prescriptions;
> the capture, parity, and env-pinning **principles** survive, applied to the live
> surfaces — roster, chat transcript, board, office scene, agent console. If a reference
> file still smells of the old vocabulary, read its principle and ignore its nouns.

---

## 1. Core rules and the two lanes

**Never kill the operator's live Launcher session to take a screenshot or to unblock a
build.** (Live incident 2026-07-25: `open_app_tab(reap_stale=true)` name-reaped the
operator's running Launcher mid-conversation, and the Hermes serve child executing the
requesting agent's own turn died with it. The reap is manifest-scoped now — it can only
touch PIDs recorded in QA launch manifests — but the workflow rule stands.)

Choose the lane by what the ask actually needs.

**Lane A — pure capture.** "Screenshot the current app / what's on screen now."
Message the existing QA persona/session through `hermes harness mission-chat message` and
have that admitted QA turn call `mcp_launcher_qa_screenshot_window` directly. That tool is
a pure capture primitive — no launch, no attach, no login, no reap — and it captures the
operator's live window by title prefix (`Eternia Launcher` matches both the normal title
and the `(stagec-smoke)` one). Do not substitute a generic MCP client or a helper process,
and do not ask QA to run `open_app_tab` / `launch_or_attach` first: those controls cannot
attach to a user-launched Launcher and will spawn a second instance.

**Lane B — driven proof.** Navigate, click, log in, fill a form, verify state, then
capture. This needs a QA-profile (`stagec-smoke`) marionette launch — a separate logged-in
identity, running side by side with any user session. Default `reap_stale:false`; pass
`reap_stale:true` only to clear stale QA corpses from earlier QA launches.

Standing rules that apply to both lanes:

- Semantic controls only. Coordinate clicks, window resizing, and overview screenshots are
  debug fallbacks, never acceptance evidence. If a human-visible control is not in the
  registry, record the semantic-control gap.
- Screenshot beats semantics for user-facing claims. If `get_buttons()` says `selected=true`
  and the pixels disagree, the pixels win and the disagreement is a parity bug, not a PASS.
- Do not accept `click_button` alone as visual proof. Verify with a state readback, then
  capture.
- Fullscreen/maximize the Launcher before any screenshot or recording.
- PNG (or a sampled key frame) for model/vision analysis; MP4 for humans, unless the defect
  is motion/timing.
- When a card or the operator gives you a PNG path as evidence, treat the path as the
  handle. Do not read or base64-load the bytes unless visual inspection is actually
  required.
- Do not start by filesystem-searching helper scripts or hand-driving unrelated PS1
  flows. Helpers are bounded MCP invocation wrappers, used only when the first-class
  in-chat tools are missing or stale.

---

## 2. The direct-existing-chat route

**ONE-OFF QA PROOF: DIRECT EXISTING CHAT ONLY.**

There is no heavier route to go looking for. The mission-task/graph/worker route no longer
exists: no `--allow-mission-goal`, no task to create/resume/unblock/tick, no worker to
nudge, no goal to spawn. And do not spawn a new persona instance merely to call
`launcher_qa` — reuse the existing visible QA instance.

The verified route is the Mission Control normal-chat layer, not a generic MCP client and
not the core `hermes -p launcher-qa chat` entry point:

1. Boot the normal Launcher and let the cockpit settle.
2. Inspect the current level/roster first with `hermes harness snapshot --json`. Select the
   existing visible QA persona instance and reuse its `default_chat_session_id`; do not
   create another instance.
3. Record the configured/persisted QA binding with `hermes harness agent list --json` and
   inspect the exact instance with `hermes harness persona-instance detail <instance>
   --json`. Do not mutate the binding during an identity probe; the live message admission
   receipt in the next step is authoritative for tool availability.
4. Send a no-tool identity/schema turn through `hermes harness mission-chat message` with
   explicit `--persona qa`, `--persona-instance-id`, `--session-id`, and a unique
   `--client-message-id`. Require `profile_timing.mcp_admitted_servers=1`, the full Launcher
   QA tool set loaded (26 tools at last count), zero calls spent, and `run_ids: []`.
5. Send one non-mutating semantic MCP request through the same chat. Require a real chat
   tool receipt plus `mcp_calls_spent=1` before asking for navigation or pixels.
6. Only then request the bounded QA action. For screenshots, report only an actual PNG and
   reproduce its `MEDIA:<absolute path>` line verbatim.

**The chat lane does admit MCP.** Do not repeat the old claim that Mission Control chat
cannot admit MCP or that Stage C cannot be driven from a chat turn. That gap was
root-caused and fixed 2026-08-26: the serve venv was missing the `mcp` pip extra. The
direct route works.

Do not misread the static `persona-instance detail` preview either. It renders the
`harness` preview lane and may say `mcp_not_registered_on_lane`; the actual `mission-chat
message` turn performs MCP admission/discovery.

Verified live 2026-07-28 using existing `personainst_qa` and session
`persona_chat_personainst_qa_469e5554a197`: identity turn
`stagec-direct-route-identity-20260728-1633` admitted one server and loaded 26 tools; the
semantic turn `stagec-direct-runtime-state-20260728-1635` spent exactly one MCP call, and
`mcp_launcher_qa_get_runtime_state` succeeded. It attached to PID 26544 via
`direct_control`, entrypoint `lib/main_marionette.dart`, Launcher credential profile
`stagec-smoke`, Harness root `.hermes/agent-runtime` resolved from `env`. The Hermes agent
profile (`launcher-qa`) and the Launcher credential profile (`stagec-smoke`) are distinct
and both must be reported accurately.

### Bounded MCP-helper fallback

Use this only when the existing QA chat's MCP admission or loaded wrapper schema is
unavailable/stale **and** the exact gap has been recorded. It is not the default route and
must not be used to bypass inspecting and reusing the existing QA persona/session. From
repo root `X:\Unreal Engine\Engine\Launcher\EterniaLauncher`:

```bash
cd 'X:/Unreal Engine/Engine/Launcher/EterniaLauncher'

powershell.exe -NoProfile -ExecutionPolicy Bypass \
  -File docs/stages/qa-reboot/scripts/Invoke-LauncherQaMcpTool.ps1 \
  -Tool mcp_launcher_qa_open_app_tab \
  -ArgsJson '{"tab":"library","browser_login":true,"credential_profile":"stagec-smoke","screenshot":false,"reap_stale":false}' \
  -CallTimeoutSeconds 240

powershell.exe -NoProfile -ExecutionPolicy Bypass \
  -File docs/stages/qa-reboot/scripts/Invoke-LauncherQaMcpTool.ps1 \
  -Tool mcp_launcher_qa_screenshot_window \
  -ArgsJson '{"window_title_prefix":"Eternia Launcher","label":"alice_library_current","out_dir":"X:/tmp/stagec/screenshots"}' \
  -CallTimeoutSeconds 120
```

The helper is still an MCP path — it launches and calls the Stage C MCP server with safe
timeouts and envelopes. Do not call it "not MCP". Its one real limitation is that it starts
a fresh server lifetime per call, so a follow-up `get_buttons`/`click_button` can return
`app_not_attached`; that is a helper session boundary, not proof the app cannot be driven.

---

## 3. Launch / attach and env pinning

### `mcp_launcher_qa_open_app_tab`

The composed entry point: launch or attach, authenticate, navigate, and optionally
screenshot in one call. It owns the composed screenshot knobs (`screenshot: true` plus the
internal-fallback behavior); `screenshot_window` does not.

Important args:

- `tab` — requested surface, e.g. `library`, `news`, `posts`, `shop`, `downloads`, `ai`,
  `missionControl`.
- `browser_login: true` — allow the browser-login fallback if needed.
- `credential_profile: stagec-smoke` — use the Stage C smoke credentials/profile.
- `screenshot: false` when you intend to call the primitive screenshot tool afterward.
- `reap_stale: false` by default — attach to a known healthy QA session when one exists.
  Pass `true` only when a stale/broken QA launch must be cleared; the reap is
  manifest-scoped (only PIDs in `direct_exe_*\_active\pid_*.json` QA launch manifests) and
  never touches a user-launched Launcher.
- `hermes_profile`, `harness_runtime_root`, `hermes_home` — env pins, see below.

Acceptance fields before you interact or capture:

- `ok=true`
- `app.attached=true`
- `app.window_visible=true`
- `app.qa_control_ready=true`
- `auth_state.status=authenticated`
- `navigation_state.selected_tab=<requested tab>`
- `navigation_state.blocking_modal_present=false`

### Env pinning is mandatory for parity proof

**Always pass explicit `hermes_profile`, `harness_runtime_root`, and `hermes_home`** on
`open_app_tab` / `launch_or_attach` smoke runs, then verify the launch envelope reports
`app.hermes_profile` non-null, `app.harness_runtime_root_configured:true`, and
`app.hermes_home_configured:true`.

If the envelope comes back unpinned (`hermes_profile:null` or
`harness_runtime_root_configured:false`), the screenshot or video is **invalid** as smoke
proof even if navigation and auth succeeded. Rerun pinned rather than diagnosing UI state.

Then distinguish three truth states before interpreting an empty panel:

1. **Real empty** — the pinned root genuinely has nothing, matching `hermes harness persona
   list` / `board list` from the same root.
2. **Bridge unavailable** — the Launcher could not read/decode the snapshot; the UI should
   show a bridge diagnostic, not an empty state.
3. **Parity mismatch** — the CLI reports rows for the same pinned root but the Launcher maps
   none. Render/report a bridge diagnostic; do not accept the empty UI as proof.

If the operator's normal Launcher shows content but pinned smoke does not, compare the
exact Harness store/root/profile for normal vs smoke first — that is usually a
root/profile mismatch, not a mapping regression. If the helper accepts pin flags but the
envelope still shows unpinned fields, audit the MCP launch manager/composer
(`buildLaunchArgs`, open-tab launch composition) for missing argument forwarding before
touching UI mapping. Post-fix, preserve the guardrail that a pinned request refuses an
unpinned live disk session with a bounded `app_env_pin_mismatch` instead of silently
attaching. See `references/mission-control-launcher-qa-smoke-env-parity.md`.

### Session persistence and freshness

- The QA window is **persistent** (2026-07-25 job-escape): it survives MCP server exits, and
  `launch_or_attach`/`open_app_tab` re-attach to it across calls (`attached_from_disk:true`,
  same PID). Do not treat a still-running QA window from a previous call as stale — attach
  to it. If its runtime gate fails, the tools self-heal (reap that instance and relaunch)
  automatically; `self_heal_relaunch:true` in the envelope diagnostics says that happened.
- Stage C rejects a regular `flutter build windows --debug` binary with
  `launch_wrong_debug_target_missing_marionette`. That is a build-target guardrail, not a
  product bug. Rebuild the QA target:
  `flutter build windows --debug --target lib/main_marionette.dart`.
- If that rebuild fails because `WebView2Loader.dll` is locked by the operator's normal
  `eternia_launcher`, **do not close or kill it automatically.** Record the exact PID and
  locked output as the freshness blocker and ask the operator to close it or authorize an
  isolated QA build output. (Verified 2026-07-28: PID 36404 held the lock; the run produced
  only a failed log, no envelope and no PNG.)
- Rebuild the canonical `tool/stagec_qa_mcp_server/build/stagec_qa_mcp_server.exe` after
  source or tool-catalog changes, or Hermes MCP discovery will not list new tools.
- In-chat MCP tools can be stale if they were loaded before an MCP server rebuild. If a
  fresh helper sees a new tool but the chat namespace does not, report a host/session reload
  need and continue with the bounded helper.

---

## 4. Semantic controls and navigation

### Schema first, always

Before calling any page-local tool, check what the live schema and registry actually
expose. `get_buttons()` and the tool schema are the contract; the visible UI is not.

- `get_buttons(include_disabled=true, include_invisible=true)` unfiltered when you are
  discovering; scoped (`get_buttons(scope="...")`) once you know the scope exists.
- If a scoped query is rejected by an already-loaded chat tool schema, unfiltered
  `get_buttons()` plus id-based `click_button` still works — id-clicks do not need the new
  scope enum. Report that a Hermes/MCP reload is needed for scoped queries; do not
  coordinate-click.
- If `get_buttons()` reports `registry_dropped_reasons: unknown_scope`, three layers must be
  updated together: the app marionette hook, the in-app `StageCQaCommandBus` known
  scope/kind allowlists, and the Stage C MCP server tool schema enum.
- If a page visibly has controls that the registry does not expose, that is a
  semantic-control gap. Record it. Do not coordinate-click it.

### Buttons

Click by stable `id`, never by coordinate. After a click, read state back
(`get_buttons`, `get_widget_state`, `get_navigation_state`) before claiming anything, then
screenshot when the visual matters. `command_unavailable` for an unsupported target is an
affordance/schema mismatch, not a mysterious app failure.

Verb-explicit naming is a contract, not a style preference: when a visible control has
multiple plausible meanings, the id must encode the verb (`...select.<id>` for a selector,
a separate `...open_details` for the opener). If you meet an ambiguous id, treat it as a
contract defect to fix or escalate — do not normalize it into your workflow. See
`references/stagec-semantic-control-naming-and-page-local-tabs.md`.

### Tabs and page-local subtabs

- Shell tabs: `open_app_tab` / `set_tab`, verified via `navigation_state.selected_tab`.
- Page-local subtabs are their own scope. Education is the worked example and its gap is now
  closed: `get_buttons()` should expose
  `education.nav.{library,explore,studio,tutor,passport,enterprise,demand,power_user}` when
  the Education root tabs are mounted. Click those ids directly (e.g. `education.nav.explore`)
  and verify the selected tab via `get_buttons()` (`selected:true`, `enabled:false` on the
  active subtab). If Education subtabs disappear from the semantic surface again, treat it
  as a regression.
- `wait_for_state` path names are stricter than some compact widget envelopes. If an
  assertion path is missing but a direct read (e.g. `get_navigation_state`) shows the truth,
  report the path mismatch as a QA/tooling issue rather than claiming the page did not
  navigate.

### Scroll

Scroll semantically. The generic shell surface is `shell.scroll`:

- `get_buttons(scope="shell.scroll")` exposes `shell.scroll.up|down|top|bottom`, kind
  `scroll`, with safe telemetry `y`, `max_y`, `viewport_height`, `can_scroll_up`,
  `can_scroll_down`.
- `click_button(id="shell.scroll.down")` animates the active content scrollable; verify
  `state.y` changed before claiming the scroll worked.
- The `scroll` tool has a **fixed target list**. Check it before calling. Historically it
  covered only `shell`, `news.feed`, and `posts.feed` — do not invent a target that is not
  in the schema.
- `shell.scroll` moves the page column, not a panel that owns its own scrollable. If
  `shell.scroll.bottom` reaches `y=max_y` and the requested panel content is still out of
  frame, that is a page-local semantic-scroll gap: report it, pair the nearest live PNG with
  deterministic widget tests, and do not overclaim.
- If `shell.scroll` is missing on a content-rich page, that is a semantic-control
  regression — usually a stale binary. Rebuild the marionette target and relaunch.

See `references/mission-control-semantic-shell-scroll-terminal-proof.md` and
`references/stagec-mcp-scroll-click-sequencing.md`.

### Forms, text entry, and modals

- Check `navigation_state.blocking_modal_present` before driving anything; a modal silently
  eats clicks. The `blocking_modal` scope carries its dismissal controls.
- Type through registered semantic controls and verify the resulting state with
  `get_widget_state`; do not drive keystrokes at coordinates.
- After any submit, require a state readback (or a visible result in a screenshot) before
  reporting success. A queued/accepted receipt with no visible outcome is a product gap, not
  a pass.

### Library

Library is the fully worked example of the select-then-open contract.

1. `get_buttons(scope='library.item')` and pick an id starting with `library.item.select.`.
2. `click_button(id='library.item.select.<safe-id>')` — this selects/focuses only.
3. Verify selection: `selected=true` on that id, or `library_focus_changed=true` /
   `library_focus_after=<item-id>` in the click response.
4. `click_button(id='library.action.open_details')` — the separate open action.
5. Verify `get_widget_state(widget='library.details')` has `details_open=true`,
   `selected_matches_details=true`, and a matching `details_item_id_safe`.
6. Only then capture screenshot evidence.

`library.nav.left|right|up|down` moves selection/focus. `library.action.close_details`
closes. Never treat an item selector as an opener, and never claim details opened from a
click envelope alone. A `button_click_failed` from re-clicking an already-selected item is
not proof that details are broken — use the details action. If the live schema still emits
the ambiguous `library.item.<safe-id>` / `kind=item` shape, the binary is stale or the
naming defect is back; rebuild, then escalate rather than adapting to it.

Screenshot overrides semantics here too: a session once had `get_buttons` reporting
`selected=true` for one title while the visibly centered card was a different one. That is
a parity bug to route, not a pass.

See `references/stagec-library-select-vs-open-contract.md`,
`references/stagec-library-semantic-controls.md`,
`references/stagec-library-visual-semantic-parity.md`,
`references/library-mcp-semantic-action-pitfall.md`, and
`references/library-semantic-click-scroll-pitfalls.md`.

### Board / kanban lane

The board is a live surface (`hermes harness board list`), and the same discipline applies:
enumerate the board scope with `get_buttons()`, click cards and lane controls by stable id,
read the selected/moved state back, and screenshot only after the state readback. If the
board's cards or lane controls are not in the registry while a human can click them, that is
a semantic-control gap to record — not a coordinate-click opportunity.

For the process side of board-routed work — reusing prior green evidence instead of
re-running suites after compaction, preferring semantic evidence over long screenshot retry
loops, and handing off with exact commands/exit codes/artifact paths — see
`references/stagec-mcp-kanban-efficiency.md`.

### Agent Console and drawers

Cockpit drawers are page-local semantic controls under `mission_control.drawer.*`
(`agents`, `terminal`, `runtime`, `close`). Click the exact id, verify `selected=true` and
that `mission_control.drawer.close` became enabled, then capture. Persona selectors appear
as `mission_control.agent.select.<instance>` and operator-channel actions under
`mission_control.agent_chat.*`. Enumerate rather than assume a fixed list.

Acceptance rule for the console: MCP mechanics are not product success. After
`send_test_message`, the UI must show an assistant response in the same transcript, a live
queued/running/completed turn state with bounded refresh, or a precise actionable blocker.
Accepted-and-then-nothing is a runtime gap even when the click returned `ok=true`. See
`references/mission-control-drawer-semantic-controls-and-agent-console.md`,
`references/mission-control-agent-chat-mcp-controls.md`, and
`references/mission-control-persona-console-live-smoke.md`.

### The `/qa/setTab` REST fallback

If the in-chat MCP wrapper schema rejects a tab value (historically `missionControl`) but a
live QA marionette session already exposes the redaction-safe port/nonce from its launch
artifact, the bounded direct QA REST route is an acceptable navigation fallback:

- `POST http://127.0.0.1:<port>/qa/setTab`
- header `X-Stagec-Qa-Nonce: <nonce>`
- body `{ "tab": "missionControl" }`

Verify the navigation state afterwards, still use `mcp_launcher_qa_screenshot_window` for
the actual pixels, and record the wrapper/schema mismatch as a tool gap. Try the bounded
MCP helper/open-tab path first — a freshly rebuilt helper may accept a tab value that the
stale chat wrapper rejects. Never coordinate-click instead. See
`references/mission-control-direct-tab-and-delivery.md`.

---

## 5. Screenshot capture and delivery

### `mcp_launcher_qa_screenshot_window`

A primitive, not an orchestrator. It screenshots whatever the current window is showing.
Use it after navigation/click/login is already done. It has its own small arg set — the
composed screenshot knobs belong to `open_app_tab`, not here.

```json
{
  "window_title_prefix": "Eternia Launcher",
  "label": "alice_library_current",
  "out_dir": "X:/tmp/stagec/screenshots"
}
```

Acceptance fields:

- `ok=true`
- a stable top-level `image_path`
- `capture_method` / `capture_method_used` = `printwindow`
- a non-trivial `byte_count`
- `bounds.width` / `bounds.height`
- `redaction.safe=true`

If `ok=false` with `helper_window_not_found`, the app/window was not visible or not
launched. Go back to `open_app_tab`/launch — do not debug the screenshot tool first.

Validated live 2026-05-18: `open_app_tab(library, browser_login=true,
credential_profile=stagec-smoke, screenshot=false)` returned `ok=true`,
`auth_state.status=authenticated`, `navigation_state.selected_tab=library`,
`window_visible=true`, `qa_control_ready=true`; then `screenshot_window` returned
`capture_method=printwindow`,
`image_path=X:\tmp\stagec\screenshots\alice_library_current_20260518103646064.png`,
`byte_count=1076666`, `1266x793`, redaction safe.

### Capture sequence

1. Launch/attach with MCP (pinned).
2. Log in with MCP if needed.
3. Navigate to the requested tab/page with MCP.
4. Scroll semantically to the exact requested panel.
5. Foreground and maximize/fullscreen the window.
6. Capture the current visible window with the primitive screenshot tool.
7. Deliver `MEDIA:<absolute-path>` unless analysis was requested.

### Exact panel or report the gap

When the operator asks for a specific panel — the roster, the chat transcript, a board
lane, the office scene, the agent console — the deliverable is a readable PNG of **that**
panel. Scroll to it and verify the PNG visibly contains it before sending. Do not
substitute an overview, a header region, or a window-resize workaround. If the target
cannot be exposed, capture the nearest supported surface as explicitly partial evidence,
pair it with deterministic widget/build proof, and report the limitation in words:

```text
Live Stage C shell screenshot captured, but exact <panel> screenshot QA was not performed
because the current MCP semantic surface does not expose a direct route/control for <panel>.
```

See `references/mission-control-exact-panel-screenshot-fallback.md`.

### Blank and white captures

Never send blank proof. Classify it instead:

- `low_information_capture` can be **honest**. The `stagec-smoke` account follows no
  communities, so the Posts **For You** feed is a genuinely near-empty dark page and the
  gate correctly refuses it (~95% blank pixels on a real render). For Posts proof, navigate
  to **Explore** (which has content) or use the internal Flutter fallback and label it.
  Verified live 2026-07-25.
- `helper_blank_capture` after the semantic gates pass (authenticated, expected
  `selected_tab`, QA control ready, no blocking modal) → use the Marionette internal Flutter
  screenshot fallback: extract the VM service URI from the child stdout log, call
  `ext.flutter.marionette.takeScreenshots` as a **direct extension path**, decode the base64
  PNG, sanity-check it, and label it as internal Flutter screenshot fallback proof. Do not
  use the `callServiceExtension?...method=...` shape — it can report `Method not found` even
  while the direct path works. Never let the VM service URI (it carries a local auth token)
  into envelopes, chat output, or logs. See
  `references/stagec-marionette-internal-screenshot-fallback.md`.
- PrintWindow blank but a foreground capture shows the UI → capture-method compatibility.
- Both blank → app rendering/bootstrap/window-surface failure. Switch to a known-good tab
  once; if still blank, classify it as a high-severity visual QA blocker. See
  `references/stagec-blank-visual-after-semantic-nav.md`.
- Uniform white pixels while semantics are healthy → report `live Windows pixel proof
  blocked: white render surface`. A widget/golden fallback is clearly-labeled partial
  evidence only, and block-glyph GoogleFonts text is layout-only, not readable proof. See
  `references/mission-control-terminal-screenshot-white-render.md`.

### Delivery

When the operator says "send me it", respond with the native media attachment and minimal
text:

```text
MEDIA:X:\\\\tmp\\\\stagec\\\\screenshots\\\\alice_library_current_20260518103646064.png
```

A live PNG path is enough to answer "show me how it looks". Optional golden tests and
visual-regression artifacts are **not** part of the user-facing proof contract unless
explicitly requested — do not block the response on them. If a temporary golden hits
`google_fonts`/asset trouble, stop retrying it unchanged, delete the temp artifact, deliver
the PNG, and note the validation blocker separately.

If the operator supplies Snipping Tool BMP files, convert to PNG before delivery/analysis.
PNG is the right format for UI/text screenshots; never JPEG-compress Launcher, cockpit,
code, or terminal screenshots unless lossy smaller files were explicitly asked for. See
`references/snipping-tool-bmp-conversion.md`.

For user-facing Launcher UI changes, live Stage C screenshot QA is the standard visual
proof lane when available. It must not block deterministic non-UI work, but if it is
skipped or blocked the report must say `live Stage C screenshot QA not performed` with the
blocker reason. "Rebuilt and relaunched Stage C" is not screenshot QA: the lane completes
only when a fresh PNG path is captured and reported, or the exact blocker is named. See
`references/mission-control-screenshot-qa-lane.md`.

---

## 6. Video capture

Video is **human proof by default**. Do not send full videos to model vision unless the
defect depends on motion, timing, transitions, loading, or flicker; if the question is
static, sample one key frame or capture a PNG instead.

**Preflight.** Before starting FFmpeg, verify the Launcher is already on the intended page
and the QA control path is ready. If an env-pinned launch reports `window_visible=true` but
`qa_control_ready=false`, do **not** start recording and do not present a partial attempt.
Diagnose the QA-control bind/readiness path or relaunch through the bounded MCP/open-tab
workflow, then record only once navigation and parity can be verified.

**Record the window, not the desktop.** Desktop capture will happily record a browser that
owns the foreground while the Launcher sits behind it.

```bash
ffmpeg -y -f gdigrab -framerate 15 \
  -i title='Eternia Launcher (stagec-smoke)' \
  -t 50 -c:v libx264 -preset ultrafast -pix_fmt yuv420p \
  '/c/Users/beast/AppData/Local/EterniaLauncher/stagec-smoke-local/videos/<label>_<timestamp>.mp4'
```

**Bound the duration; never force-kill.** Pass `-t <seconds>` up front so FFmpeg exits
normally and writes the container metadata. A killed FFmpeg leaves an invalid MP4 that
fails with `moov atom not found`. Verify before delivery:

```bash
ffprobe -v error -show_entries format=duration,size -of default=noprint_wrappers=1:nokey=0 '<video>.mp4'
```

**Sequence.** Maximize → 3-5s sample capture → extract one frame → inspect that frame only,
to confirm you are recording the Launcher on the right surface → bounded production capture
→ `ffprobe` → sample a key frame → deliver.

```text
MEDIA:C:\\\\Users\\\\beast\\\\AppData\\\\Local\\\\EterniaLauncher\\\\stagec-smoke-local\\\\videos\\\\launcher_<label>_<timestamp>.mp4
```

**Stale MP4s are not proof.** If the operator says the screenshot is right but the video is
not, do not defend or resend the old file. Treat it as stale, re-record the current
fullscreen window, sample a key frame, verify that frame shows the claimed state, then send
the fresh MP4. Delete failed desktop-capture attempts that recorded unrelated apps. See
`references/mission-control-window-title-video-capture.md` and
`references/mission-control-current-video-proof-and-env-pinning.md`.

**Artifacts must show the claimed state.** If a live chat turn, board change, or roster
change drove the claim, the pixels must show the corresponding panel state. CLI/Harness
success without matching visible evidence is not UI proof — report the Launcher/read-model
mismatch as a high-severity regression instead of presenting the artifact as success. If a
live interaction ran during capture, include only the compact final state or blocker unless
logs were requested.

---

## 7. Credential preflight and recovery

Reuse the existing Stage C smoke credential path: `credential_profile: stagec-smoke`,
`browser_login: true`. Do not invent a new credential system and do not ask the operator
for credentials unless the existing Stage C smoke profile fails. The Launcher should
display only redaction-safe auth state — `authenticated` / `login_required` / `failed`.

On `browser_login_helper_failed` / `auth_secret_unavailable`, distinguish **credential
contract existence** from **active runner access**. The Stage C credential path may be
fully provisioned (`stagec-smoke`, username `qa-stagec-smoke`, k8s
`eternia-staging/stagec-smoke-credentials`, Windows Credential Manager
`EterniaStageC/stagec-smoke`) while the current runner cannot reach it because kube auth
expired, the Credential Manager fallback is absent for this Windows user, or the
interactive fallback is unavailable in a noninteractive helper.

Report: "the credential path exists and is provisioned; the current runner cannot access a
source." Never "we have no credentials."

Safe diagnosis, in order:

```bash
command -v kubectl
kubectl config current-context
kubectl auth can-i get secret/stagec-smoke-credentials -n eternia-staging
kubectl -n eternia-staging get secret stagec-smoke-credentials -o name
```

Healthy output is `local`, `yes`, `secret/stagec-smoke-credentials`, with keys `password
username`. The kubeconfig lives at `C:\Users\beast\.kube\config` (`/c/Users/beast/.kube/config`
from Git Bash) and resets roughly every two weeks. `Unauthorized` means a stale kube
token/session. `x509: certificate signed by unknown authority` after a Rancher-generated
config rewrite means the embedded CA does not match the served Rancher proxy cert; the
pragmatic in-session fix is:

```bash
kubectl config set-cluster local --insecure-skip-tls-verify=true
```

**Never print secret `.data` values or decoded credentials** — report secret names and key
names only. See `references/stagec-smoke-kubeconfig-credential-preflight.md` and
`references/stagec-smoke-credential-source-vs-runner-access.md`.

---

## 8. Linked references

**Capture policy and delivery format**

- `references/fullscreen-screenshot-and-video-evidence-policy.md` — maximize/fullscreen the
  Launcher before screenshots and recordings; PNG/key-frame for model analysis, MP4 as human
  proof unless motion/timing is the defect.
- `references/snipping-tool-bmp-conversion.md` — convert Snipping Tool BMP screenshots to
  PNG for delivery and vision/QA; JPEG only for photo-like/lossy cases explicitly requested.
- `references/mission-control-screenshot-qa-lane.md` — user-facing Launcher UI changes need
  live Stage C PNG proof when available; if skipped or blocked, explicitly report `live
  Stage C screenshot QA not performed` and the reason, with a triage table for the blocker
  classes.
- `references/mission-control-direct-tab-and-delivery.md` — when the MCP tab enum wrapper is
  stale, use the redaction-safe QA control REST `/qa/setTab` path with an existing
  port/nonce, capture with `screenshot_window`, deliver the PNG immediately, and never let
  optional golden validation block a screenshot request.
- `references/mission-control-exact-panel-screenshot-fallback.md` — if MCP can launch and
  capture the shell but the exact panel is not semantically exposed, capture the nearest
  supported evidence, pair it with deterministic tests, and report the exact-panel
  limitation instead of coordinate-clicking.

**Blank, white, and fallback captures**

- `references/stagec-blank-visual-after-semantic-nav.md` — when semantic state says mounted
  and navigated but screenshots are blank, separate capture-method flake from a true
  rendered-visual blocker and report visual QA as failed until pixels prove otherwise.
- `references/stagec-marionette-internal-screenshot-fallback.md` — when Stage C gates pass
  but Win32/PrintWindow returns blank pixels, call the live VM-service
  `ext.flutter.marionette.takeScreenshots` direct extension path, decode the base64 PNG, and
  label it as internal Flutter screenshot fallback proof.
- `references/mission-control-terminal-screenshot-white-render.md` — semantic state can be
  healthy while Windows pixels are a uniform white render surface; do not send blank proof,
  classify the blocker, and treat block-glyph golden fallback as layout-only.
- `references/posts-algorithm-controls-visual-proof.md` — reject stale/empty/loading/overflow
  screenshots, rebuild stale binaries, degrade read-only config hydration to a fallback on
  backend HTML/500, and require the visible controls with no Flutter overflow banner before
  attaching screenshot proof.

**Video**

- `references/mission-control-window-title-video-capture.md` — capture the Launcher by exact
  window title with `gdigrab` so desktop capture cannot record the browser instead; bound the
  duration, never force-kill FFmpeg, verify with `ffprobe`, sample a frame.
- `references/mission-control-current-video-proof-and-env-pinning.md` — if a screenshot shows
  the current state but a sent video does not, treat the MP4 as stale, re-record the exact
  fullscreen window, verify a key frame, and require env-pinned launch metadata before
  sending.

**Launch, env pinning, and semantic controls**

- `references/mission-control-launcher-qa-smoke-env-parity.md` — pin the Launcher child
  process to the intended Harness runtime root, audit the whole launch chain when pins do not
  land, and distinguish real-empty from bridge-unavailable from parity-mismatch.
- `references/mission-control-semantic-shell-scroll-terminal-proof.md` — the `shell.scroll.*`
  contract: verify the controls, scroll by semantic button, prove `state.y` changed, preserve
  the session for the screenshot, and rebuild `lib/main_marionette.dart` if the running
  binary lacks the scroll observer.
- `references/mission-control-drawer-semantic-controls-and-agent-console.md` — the
  `mission_control.drawer.*` contract: click the exact id, verify selected state, capture the
  Agent Console drawer, and the DM-stack layout/selection pitfalls from its migration.
- `references/mission-control-agent-chat-mcp-controls.md` — expose selected Operator Channel
  actions under `mission_control.agent_chat`, update app/bus/MCP schema allowlists together,
  account for stale loaded tool schemas, and treat "MCP click queued" as a product gap until
  a response, running state, or blocker appears.
- `references/mission-control-persona-console-live-smoke.md` — live smoke for "can I talk to
  Neko": rebuild the marionette if needed, open the Agent Console, verify persona selectors
  plus `mission_control.agent_chat` controls, click `send_test_message`, and state the
  boundary versus a full freeform message/response proof.
- `references/mission-control-harness-attachment-contract-and-ui-proof.md` — the structured
  image/video/file attachment-handle shape and its capability allowlist, the Launcher
  test patterns around it, and the honest non-AAA visual proof rule.
- `references/stagec-mcp-helper-boundary-current-window-screenshot.md` — the bounded operator
  helper is an MCP path, not a PowerShell replacement for MCP; the correct end-to-end
  open-tab-then-screenshot sequence through it.
- `references/stagec-mcp-kanban-efficiency.md` — lean board/kanban execution: semantic
  evidence over screenshot retry loops, reuse of prior green evidence after compaction, and
  handoffs carrying exact commands, exit codes, and artifact paths.

**Library and page-local control contracts**

- `references/stagec-library-select-vs-open-contract.md` — the authoritative select-vs-open
  contract: `library.item.select.<safe-id>` selects, `library.action.open_details` opens,
  and `library.details` state proves it.
- `references/stagec-library-semantic-controls.md` — the Library item/navigation control
  surface and the validated operator smoke, deferring to the select-vs-open contract for id
  shape.
- `references/stagec-library-visual-semantic-parity.md` — when semantic state and screenshot
  evidence disagree, the screenshot wins for user-facing claims and the disagreement is a
  parity bug to route.
- `references/stagec-semantic-control-naming-and-page-local-tabs.md` — the verb-explicit id
  rule and the page-local subtab gap pattern (Education is the worked example).
- `references/stagec-mcp-scroll-click-sequencing.md` — schema-first scroll/click sequencing
  and Library select-then-open verification.
- `references/library-mcp-semantic-action-pitfall.md` — a Library item click may select
  rather than open; enumerate controls, use the explicit details action, verify
  `library.details`, and respect the scroll target schema.
- `references/library-semantic-click-scroll-pitfalls.md` — the same pitfall from the
  interaction-sequence angle, with the reporting discipline for unsupported scroll targets.

**Credentials**

- `references/stagec-smoke-kubeconfig-credential-preflight.md` — the credential preflight for
  `auth_secret_unavailable`: verify the Windows kubeconfig, Rancher auth/TLS, and secret
  visibility, and apply the pragmatic `insecure-skip-tls-verify` recovery without printing
  secrets.
- `references/stagec-smoke-credential-source-vs-runner-access.md` — distinguish provisioned
  Stage C agent credentials from the active runner being unable to reach k8s or Credential
  Manager; report the latter as credential-access blocked, not "credentials do not exist."
