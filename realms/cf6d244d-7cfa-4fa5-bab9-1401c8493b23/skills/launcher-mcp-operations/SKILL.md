---
name: launcher-mcp-operations
description: "General-purpose Eternia Launcher MCP operation skill: drive, verify, and capture the Launcher through the launcher_qa MCP surface — sign-in first (dev_login), launch/attach and env pinning, semantic controls and navigation, forms/scroll/tabs, library and board lanes, in-app screenshots, credential preflight, and evidence delivery."
version: 2.1.0
metadata:
  hermes:
    surfaces: [mission_chat]
    modes: [standard]
    load_policy: recommended
---

# Launcher MCP Operations

How to operate the Eternia Launcher end-to-end through the `launcher_qa` MCP surface:
launch or attach it, sign it in, navigate it, click its semantic controls, read its state
back, and capture its own rendered frame as evidence. Screenshots are one lane in here, not
the whole skill.

> **Source of truth.** This package is authored in the EterniaLauncher repository at
> `agent_prompts/skills/launcher-mcp-operations/` and reaches
> `$HERMES_HOME/shared/skills/launcher-mcp-operations/` by promotion and realm publish
> (see `agent_prompts/skills/README.md` there). An edit made only in the live copy is overwritten
> by the next promotion or realm pull.

> **Historical-vocabulary note (2026-07-30, rewritten 2026-08-28):** the harness
> goal/task mission lane was removed
> (`hermes-agent/docs/agent-runtime-harness/archive/2026-08-22-pre-consolidation/16-mission-lane-removal.md`).
> There are no goals, missions, runs, proof gates, task ids, `run-until-settled`, or
> Active Missions counts any more, and the projector/read-model is retired. Chat is the
> only lane. If a reference file still smells of the old vocabulary, read its principle
> and ignore its nouns.

---

## 0. First step: sign in before any signed-in tab

**Call `mcp_launcher_qa_dev_login` before you open any surface that needs a signed-in
session** — or pass `dev_login: true` to `mcp_launcher_qa_open_app_tab`, which does the
same inside the call. A `stagec-smoke` Launcher is a GUEST until you do.

- Why first: on 2026-10-02 an agent opened News on a signed-out QA Launcher; the call
  rebuilt and launched for 220 s and then refused `auth_required`. One `dev_login: true`
  on that call would have made it one call that succeeded.
- `dev_login` signs in as the dev player in ~0.15 s, with no browser and no access token.
  It exists only in QA builds (debug marionette and the fast QA lane); elsewhere it
  answers `dev_login_unavailable`, never a false ok.
- What needs it: the app says, not this skill. `open_app_tab` asks the app — the
  `shell.nav.<tab>` control reports `disabled_reason: needs_account` for a tab the app
  gates on an account (today: `studio`). Every other tab opens for a guest in its
  signed-out shape, with a warning in the envelope. If you want the signed-in content of
  any surface (entitlements, account, Studio), sign in first.
- `dev_login` signs the shell in but mints no token, so backend-backed DATA still reads
  "Session expired". When real backend data is the point, use the staging credential flow
  instead (`browser_login: true`, section 7).
- `dev_login` resets the shell to Home on the signed-in edge. Sign in, THEN navigate.

---

## 1. Core rules and the two lanes

**The QA Launcher never covers the owner's screen** (owner ruling
OR-2026-10-02-qa-never-covers-the-screen). It does not open on top of other windows, take
focus, restore or foreground itself, or open maximized over the owner's work. Every
screenshot is rendered INSIDE the app from its own frame (the `captureFrame` verb). Never
foreground, focus, restore or maximize the window, and never take a desktop capture —
not PrintWindow, not `CopyFromScreen`, not a Snipping Tool grab, not an FFmpeg window
grab. A capture the app cannot render fails with a named reason; report it.

**Never kill the operator's live Launcher session to take a screenshot or to unblock a
build.** (Live incident 2026-07-25: `open_app_tab(reap_stale=true)` name-reaped the
operator's running Launcher mid-conversation, and the Hermes serve child executing the
requesting agent's own turn died with it.) Leave `reap_stale` false; self-heal already
ends a broken QA instance by pid.

Choose the lane by what the ask actually needs.

**Lane A — pure capture.** "Screenshot the QA Launcher as it is now." Have the admitted QA
turn call `mcp_launcher_qa_screenshot_window` directly. It is a pure capture primitive — no
launch, no login, no reap — that renders the attached QA session's current frame. It
cannot capture a user-launched Launcher (that binary has no QA control surface) or any
other window (`capture_foreign_window_refused`).

**Lane B — driven proof.** Sign in, navigate, click, fill a form, verify state, then
capture. This needs a QA-profile (`stagec-smoke`) launch running side by side with any
user session.

Standing rules that apply to both lanes:

- Semantic controls only. Coordinate clicks, window resizing, and overview screenshots are
  debug fallbacks, never acceptance evidence. If a human-visible control is not in the
  registry, record the semantic-control gap.
- Screenshot beats semantics for user-facing claims. If `get_buttons()` says `selected=true`
  and the pixels disagree, the pixels win and the disagreement is a parity bug, not a PASS.
- Do not accept `click_button` alone as visual proof. Verify with a state readback, then
  capture.
- PNG for model/vision analysis and for human proof. There is no in-app video lane; see
  section 6.
- When a card or the operator gives you a PNG path as evidence, treat the path as the
  handle. Do not read or base64-load the bytes unless visual inspection is actually
  required.
- The agent path is MCP-only. Do not filesystem-search helper scripts or drive PowerShell
  QA scripts; those are operator/CI tools.

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
   `--client-message-id`. Require `profile_timing.mcp_admitted_servers=1`, the Launcher QA
   tool set loaded, zero calls spent, and `run_ids: []`.
5. Send one non-mutating semantic MCP request through the same chat. Require a real chat
   tool receipt plus `mcp_calls_spent=1` before asking for navigation or pixels.
6. Only then request the bounded QA action — and if it touches a signed-in surface, the
   first call is `dev_login` (section 0). For screenshots, report only an actual PNG and
   reproduce its `MEDIA:<absolute path>` line verbatim.

**The chat lane does admit MCP.** Do not repeat the old claim that Mission Control chat
cannot admit MCP or that Stage C cannot be driven from a chat turn. That gap was
root-caused and fixed 2026-08-26: the serve venv was missing the `mcp` pip extra.

Do not misread the static `persona-instance detail` preview either. It renders the
`harness` preview lane and may say `mcp_not_registered_on_lane`; the actual `mission-chat
message` turn performs MCP admission/discovery.

If the chat's MCP admission or its loaded tool schema is unavailable or stale, record the
exact gap (admission receipt, missing tool, schema mismatch) and stop there. Driving the
operator's PowerShell helper scripts is not an agent fallback.

---

## 3. Launch / attach and env pinning

### `mcp_launcher_qa_open_app_tab`

The composed entry point: launch or attach, sign in if asked, navigate, and optionally
screenshot in one call. It owns the composed screenshot knob (`screenshot: true`);
`screenshot_window` does not.

Important args:

- `tab` — requested surface, e.g. `library`, `news`, `posts`, `shop`, `downloads`, `ai`,
  `missionControl`, `studio`.
- `dev_login: true` — sign in as the dev player inside this call when the Launcher is a
  guest (QA builds). The default way to reach a signed-in surface.
- `browser_login: true` with `credential_profile: stagec-smoke` — the staging
  real-credential PKCE flow, when real backend data is the point. Exclusive with
  `dev_login`.
- `screenshot: false` when you intend to call the primitive screenshot tool afterward.
- `reap_stale: false` — always. Self-heal already replaces a broken QA instance by pid.
- `hermes_profile`, `harness_runtime_root`, `hermes_home` — env pins, see below.

Acceptance fields before you interact or capture:

- `ok=true`
- `app.attached=true`
- `app.qa_control_ready=true`
- `auth_state.status=authenticated` when the surface needs a signed-in session; for a
  guest, `auth_state.sign_in_required=false` and the warning say the tab opened in its
  signed-out shape
- `navigation_state.selected_tab=<requested tab>`
- `navigation_state.blocking_modal_present=false`

An `auth_required` refusal names the tab, the app's reason, and how long the call spent
(`timing.elapsed_ms`, `timing.launch_or_attach_ms`). Retry with `dev_login: true`.

The first call after `main` moves may rebuild the isolated QA copy (minutes, not seconds).
That cost is paid once; sign in on that same call rather than paying it for a refusal.

### Env pinning is mandatory for parity proof

**Always pass explicit `hermes_profile`, `harness_runtime_root`, and `hermes_home`** on
`open_app_tab` / `launch_or_attach` smoke runs, then verify the launch envelope reports
`app.hermes_profile` non-null, `app.harness_runtime_root_configured:true`, and
`app.hermes_home_configured:true`.

If the envelope comes back unpinned (`hermes_profile:null` or
`harness_runtime_root_configured:false`), the screenshot is **invalid** as smoke proof
even if navigation and auth succeeded. Rerun pinned rather than diagnosing UI state.

Then distinguish three truth states before interpreting an empty panel:

1. **Real empty** — the pinned root genuinely has nothing, matching `hermes harness persona
   list` / `board list` from the same root.
2. **Bridge unavailable** — the Launcher could not read/decode the snapshot; the UI should
   show a bridge diagnostic, not an empty state.
3. **Parity mismatch** — the CLI reports rows for the same pinned root but the Launcher maps
   none. Render/report a bridge diagnostic; do not accept the empty UI as proof.

If the operator's normal Launcher shows content but pinned smoke does not, compare the
exact Harness store/root/profile for normal vs smoke first — that is usually a
root/profile mismatch, not a mapping regression. A pinned request refuses an unpinned live
disk session with a bounded `app_env_pin_mismatch` instead of silently attaching. See
`references/mission-control-launcher-qa-smoke-env-parity.md`.

### Session persistence and freshness

- The QA window is **persistent**: it survives MCP server exits, and
  `launch_or_attach`/`open_app_tab` re-attach to it across calls (`attached_from_disk:true`,
  same PID). Do not treat a still-running QA window from a previous call as stale — attach
  to it. If its runtime gate fails, the tools self-heal (end that instance by pid and
  relaunch); `self_heal_relaunch:true` in the envelope diagnostics says that happened.
- Stage C rejects a regular debug binary with
  `launch_wrong_debug_target_missing_marionette`. That is a build-target guardrail, not a
  product bug; `launch_or_attach` builds its isolated QA copy when its own copy is stale.
- If a build fails because a file is locked by the operator's normal `eternia_launcher`,
  **do not close or kill it.** Record the exact PID and locked output as the freshness
  blocker and ask the operator.
- In-chat MCP tools can be stale if they were loaded before an MCP server rebuild. If the
  chat namespace lacks a tool or argument this skill names, report a host/session reload
  need.

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
the actual pixels, and record the wrapper/schema mismatch as a tool gap. Try
`open_app_tab` first — a freshly reloaded tool schema may accept a tab value that the
stale chat wrapper rejects. Never coordinate-click instead. See
`references/mission-control-direct-tab-and-delivery.md`.

---

## 5. Screenshot capture and delivery

### The in-app capture lane

Every screenshot is the app's own rendered frame. `mcp_launcher_qa_screenshot_window`,
`mcp_launcher_qa_capture_screenshot` and `open_app_tab(screenshot: true)` call the
`captureFrame` verb: the root layer tree is rasterized offscreen at the window's device
pixel ratio and written to a PNG. The window is never restored, raised, focused or
maximized; it may sit behind other windows, the display may be asleep and the session
locked. There is no desktop fallback, by ruling.

### `mcp_launcher_qa_screenshot_window`

A primitive, not an orchestrator. It renders whatever the attached session is showing. Use
it after sign-in/navigation/clicks are done. It has its own small arg set — the composed
knobs belong to `open_app_tab`.

```json
{
  "label": "alice_library_current"
}
```

Acceptance fields:

- `ok=true`
- a stable top-level `image_path`
- method `in_app_frame`
- physical `width` / `height` and `device_pixel_ratio`
- the content gate's verdict (`blank_pixel_ratio`, `unique_color_count`) in
  `capture_response_safe`
- `redaction.safe=true`

`window_title_prefix` is accepted only when it names the Launcher, and is answered with
the session's own frame; any other window is `capture_foreign_window_refused`.

### Capture sequence

1. Launch/attach with MCP (pinned when parity is the claim).
2. Sign in with `dev_login` if the surface needs a signed-in session (section 0).
3. Navigate to the requested tab/page with MCP.
4. Scroll semantically to the exact requested panel and read the state back.
5. Capture with the primitive screenshot tool.
6. Deliver `MEDIA:<absolute-path>` unless analysis was requested.

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

### Refused and blank captures

Never send blank proof. A blank frame comes back as a named refusal; classify it:

- `capture_low_information_frame` can be **honest**. The `stagec-smoke` account follows no
  communities, so the Posts **For You** feed is a genuinely near-empty page. For Posts
  proof, navigate to **Explore** (which has content).
- `capture_blank_frame` after the semantic gates pass (expected `selected_tab`, QA control
  ready, no blocking modal) → switch to a known-good tab once; if still refused, classify
  it as a high-severity visual QA blocker. See
  `references/stagec-blank-visual-after-semantic-nav.md`.
- `capture_frame_failed` → the app could not render its own frame; report the `reason`.
- `capture_in_app_unsupported` → the binary predates the lane; it needs a rebuild.
- Uniform white pixels in an accepted frame while semantics are healthy → report
  `live pixel proof blocked: white render surface`. A widget/golden fallback is
  clearly-labeled partial evidence only. See
  `references/mission-control-terminal-screenshot-white-render.md`.

None of these is answered by raising the window, a desktop capture, or the VM-service
`takeScreenshots` path — all retired.

### Delivery

When the operator says "send me it", respond with the native media attachment and minimal
text:

```text
MEDIA:<absolute path from image_path>
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

## 6. Video

**There is no compliant video lane today.** A window-title FFmpeg grab records whatever is
on top of that screen region, so it only works with the QA Launcher over the owner's work —
which the ruling forbids. Do not record the desktop, and do not raise the window to make a
recording work.

When the defect depends on motion, timing, transitions, loading or flicker, capture an
in-app frame sequence — several `screenshot_window` calls, each paired with the state
readback that frame claims — and say plainly that video proof was not taken because the QA
lane has no in-app recorder.

**Stale artifacts are not proof.** An MP4 or PNG captured before the current fix/state is
invalid proof even if it is valid media. Re-capture after the change; never defend or
resend the old file. See `references/mission-control-current-video-proof-and-env-pinning.md`.

**Artifacts must show the claimed state.** If a live chat turn, board change, or roster
change drove the claim, the pixels must show the corresponding panel state. CLI/Harness
success without matching visible evidence is not UI proof — report the Launcher/read-model
mismatch as a high-severity regression instead of presenting the artifact as success.

---

## 7. Credential preflight and recovery

For a signed-in shell, `dev_login` (section 0) is the default and needs no credentials.
Use the staging credential path only when real backend data is the point:
`credential_profile: stagec-smoke`, `browser_login: true`. Do not invent a new credential
system and do not ask the operator for credentials unless the existing Stage C smoke
profile fails. The Launcher should display only redaction-safe auth state —
`authenticated` / `login_required` / `failed`.

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
username`. The kubeconfig lives at `%USERPROFILE%\.kube\config` (`~/.kube/config` from Git
Bash) and resets roughly every two weeks. `Unauthorized` means a stale kube token/session.
`x509: certificate signed by unknown authority` after a Rancher-generated config rewrite
means the embedded CA does not match the served Rancher proxy cert; the pragmatic
in-session fix is:

```bash
kubectl config set-cluster local --insecure-skip-tls-verify=true
```

**Never print secret `.data` values or decoded credentials** — report secret names and key
names only. See `references/stagec-smoke-kubeconfig-credential-preflight.md` and
`references/stagec-smoke-credential-source-vs-runner-access.md`.

---

## 8. Linked references

**Capture policy and delivery format**

- `references/in-app-capture-and-evidence-policy.md` — the ruling, how `captureFrame`
  works, what is never done (foreground, maximize, desktop capture), and the evidence
  defaults.
- `references/snipping-tool-bmp-conversion.md` — convert operator-supplied Snipping Tool
  BMP screenshots to PNG for delivery and vision/QA.
- `references/mission-control-screenshot-qa-lane.md` — user-facing Launcher UI changes need
  live Stage C PNG proof when available; if skipped or blocked, explicitly report `live
  Stage C screenshot QA not performed` and the reason, with a triage table for the blocker
  classes.
- `references/mission-control-direct-tab-and-delivery.md` — when the MCP tab enum wrapper is
  stale, the redaction-safe QA control REST `/qa/setTab` path; deliver the PNG immediately
  and never let optional golden validation block a screenshot request.
- `references/mission-control-exact-panel-screenshot-fallback.md` — if MCP can launch and
  capture the shell but the exact panel is not semantically exposed, capture the nearest
  supported evidence, pair it with deterministic tests, and report the exact-panel
  limitation instead of coordinate-clicking.

**Refused, blank, and white captures**

- `references/stagec-blank-visual-after-semantic-nav.md` — the named capture refusals and
  how to classify each; a refusal is a visual blocker, never retried by raising the window.
- `references/mission-control-terminal-screenshot-white-render.md` — semantic state can be
  healthy while the frame is a uniform white surface; classify the blocker and treat
  block-glyph golden fallback as layout-only.
- `references/posts-algorithm-controls-visual-proof.md` — reject stale/empty/loading/overflow
  screenshots, rebuild stale binaries, and require the visible controls with no Flutter
  overflow banner before attaching screenshot proof.

**Stale artifacts and env pinning**

- `references/mission-control-current-video-proof-and-env-pinning.md` — stale PNG/MP4
  artifacts are not proof; motion defects get an in-app frame sequence; env-pinned launch
  metadata is required for parity claims.

**Launch, env pinning, and semantic controls**

- `references/mission-control-launcher-qa-smoke-env-parity.md` — pin the Launcher child
  process to the intended Harness runtime root, audit the whole launch chain when pins do not
  land, and distinguish real-empty from bridge-unavailable from parity-mismatch.
- `references/mission-control-semantic-shell-scroll-terminal-proof.md` — the `shell.scroll.*`
  contract: verify the controls, scroll by semantic button, prove `state.y` changed, preserve
  the session for the screenshot.
- `references/mission-control-drawer-semantic-controls-and-agent-console.md` — the
  `mission_control.drawer.*` contract: click the exact id, verify selected state, capture the
  Agent Console drawer, and the DM-stack layout/selection pitfalls from its migration.
- `references/mission-control-agent-chat-mcp-controls.md` — expose selected Operator Channel
  actions under `mission_control.agent_chat`, update app/bus/MCP schema allowlists together,
  account for stale loaded tool schemas, and treat "MCP click queued" as a product gap until
  a response, running state, or blocker appears.
- `references/mission-control-persona-console-live-smoke.md` — live smoke for "can I talk to
  Neko": open the Agent Console, verify persona selectors plus `mission_control.agent_chat`
  controls, click `send_test_message`, and state the boundary versus a full freeform
  message/response proof.
- `references/mission-control-harness-attachment-contract-and-ui-proof.md` — the structured
  image/video/file attachment-handle shape and its capability allowlist, the Launcher
  test patterns around it, and the honest non-AAA visual proof rule.
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

**Retired 2026-10-02** (owner ruling OR-2026-10-02-qa-never-covers-the-screen; removed
from the package): `fullscreen-screenshot-and-video-evidence-policy.md`,
`mission-control-window-title-video-capture.md`,
`stagec-marionette-internal-screenshot-fallback.md`, and
`stagec-mcp-helper-boundary-current-window-screenshot.md` (the operator PowerShell helper is
not an agent path).
