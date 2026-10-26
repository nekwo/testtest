---
name: launcher-mcp-operations
description: "General-purpose Eternia Launcher MCP operation skill: drive, verify, and capture the Launcher through the launcher_qa MCP surface — sign-in first (the persisted real stagec-smoke session; dev_login is chrome only), launch/attach and env pinning, semantic controls and navigation, forms/scroll/tabs, library and board lanes, in-app screenshots, credential preflight, and evidence delivery."
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

**A REAL sign-in is the `stagec-smoke` session, and it persists.** Check
`mcp_launcher_qa_get_auth_state` first: once the owner has signed in, every later QA
launch restores that session (stored wallet, `offline_access`, refreshed at launch) and
backend data loads. Nothing to do.

- When it reads a guest, call `mcp_launcher_qa_begin_pkce_login`. It writes the
  authorization URL to the local file you name and waits 5 minutes; tell the owner the
  file path so he signs in once in his own browser. Pass `open_browser: true` only when
  he asks the Launcher to open it for him — it opens a window on his screen. While it
  waits, `get_auth_state` reads `is_authenticating: true`; a second begin is refused
  ("A sign-in is already in progress.") and leaves that flow alone. You never type,
  read or handle a credential, and you never complete the sign-in yourself.
- `dev_login` (or `dev_login: true` on `mcp_launcher_qa_open_app_tab`) is CHROME ONLY:
  the shell shows the dev player in ~0.15 s but no token is minted, so every
  backend-backed surface (entitlements, account, feeds) still reads signed out /
  "Session expired". Use it only to look at auth-gated chrome, never as "signed in".
  It exists only in QA builds; elsewhere it answers `dev_login_unavailable`.
- What needs a signed-in session: the app says, not this skill. `open_app_tab` asks the
  app — the `shell.nav.<tab>` control reports `disabled_reason: needs_account` for a tab
  the app gates on an account (today: `studio`). Every other tab opens for a guest in its
  signed-out shape, with a warning in the envelope.
- Signing in resets the shell to Home on the signed-in edge. Sign in, THEN navigate.

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
- PNG for model/vision analysis and for human proof — the default, always. Video
  (`mcp_launcher_qa_record_video`) is opt-in and token-expensive; record only when a row or
  the operator explicitly asks for video. See section 6.
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
   first call is `get_auth_state` (section 0). For screenshots, report only an actual PNG and
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
- `dev_login: true` — show the dev player inside this call when the Launcher is a guest
  (QA builds). Chrome only: no token, backend data does not load (section 0).
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
- `content_state=ready` before you present a capture as proof of the tab's content (see
  section 5, "Settle before capture")

Every result, success or refusal, carries `stages[]` — `launch`, `sign_in`,
`tab_switch`, `settle`, `capture`, each with `status` and `elapsed_ms` — and `timing`
(`elapsed_ms`, `<stage>_ms`). Read the failed or slow stage there instead of guessing. A
client that sends `_meta.progressToken` also hears each stage start and end as an MCP
progress notification, and a heartbeat at least every 5 s while one runs.

An `auth_required` refusal names the tab, the app's reason, and how long the call spent
(`timing.elapsed_ms`, `timing.launch_or_attach_ms`). Sign in for real (section 0);
`dev_login: true` only gets past it for chrome.

**Which build QA runs (owner 2026-10-11).** With no `exe_dir`, `launch_or_attach` /
`open_app_tab` run the **Debug build of the checkout's HEAD**
(`build/windows/x64/runner/Debug`, one entry `lib/main.dart`, launched from the helper's
`DebugStageC` mirror) — never an older build: when Debug is not at HEAD the call builds
it first. The release-mode **parity build** (`ParityStageC.builds`) is the opt-in: pass
`parity: "label"` (or `"strict"`) for perf, timing and release-parity proofs. If the
owner's own Launcher runs from `Debug/`, the Debug build cannot be written and the call is
refused `launch_debug_build_locked` with the pid(s): **never close or kill that Launcher** —
report the pid and ask, or pass `parity` to use the release-mode build.

The first call after `main` moves may build the QA build — the Debug build by default, or
the isolated parity copy (usually 5-6 min) when `parity` is named. The
build is a BACKGROUND JOB, like a terminal background process: the call that needs it
starts it and returns at once with `failure_class: launch_qa_build_started` and a
`build_job` block (`job_id`, `commit`, `phase`, `started_at`, `eta_ms`); every later call
answers `launch_qa_build_rebuilding` with the same job. Neither is a fault. When the job
ends your session gets one `notifications/message` (logger
`stagec_qa_mcp_server.qa_build`, `data.event: qa_build_finished`, `outcome: ready|failed`,
`failure_tail`). Wait for it — or call `mcp_launcher_qa_build_status` with the `job_id`
and `wait_ms` (up to 240000; `timed_out: true` means still running, call again). On
`ready`, call the same tool again: it attaches to the new build (the persisted
`stagec-smoke` session restores on it — section 0). On `failed`, read `failure_tail`, fix that cause, call again. **Never
give up mid-build** and never start a build by hand to "help" it.

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
- Every Debug build of `lib/main.dart` carries the QA surface (one entry, 2026-10-11),
  dormant unless QA-launched; there is no wrong-target refusal any more.
  `launch_or_attach` builds the Debug build of HEAD when Debug is behind (or the parity
  build when `parity` is named).
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

- Shell tabs: `open_app_tab`, or the app function itself —
  `call_app_function {"method": "launcher.navigation.open", "args": {"page": "<tab>"}}` —
  verified via `navigation_state.selected_tab`. `set_tab` is retired (one action layer S2):
  opening a page is an intent, and QA reaches an intent only through the real dispatcher. A
  page this session cannot show (Studio for a guest) is refused as `app_function_refused`
  with the reason. Overlays: `launcher.navigation.open_overlay` (`account`, `apps`).
  `get_buttons` shows each control's `intent` where it is the proof of one.
- What the AGENT reads (one action layer S4): `launcher.app.about`, `.account.status`,
  `.navigation.state`, `.window.metrics`, `.voice.state`, `.media.playback.state` (every
  player with media open — post, media viewer, lesson, stream, Studio preview; the same list
  `get_media_playback_state` answers when given no `label`) — reach them with
  `call_app_function` when the proof is "the agent sees X". They share the QA probes' readers
  but are the redacted subset: `get_auth_state` (token presence), `get_navigation_state`
  (the NAMED blocking-modal kind) and `get_window_metrics` (breakpoint) stay the QA reads for
  those fields. Since S5 `launcher.navigation.state` carries `blocking_modal` (present, kind
  `dialog`/`page`, barrier) from the same route stack, and `launcher.view.content_state` is
  the agent's `get_content_state`.
- Sections and the window (one action layer S5) are intents, so a headless step calls them:
  `launcher.settings.open_section {"section": "audio"}` (wire names as in
  `account.settings.tab.<section>`), `launcher.account.open_section {"section": "wallet"}`
  (an `AccountTab` name), `launcher.window.resize {"width": 1280, "height": 800}` (floor
  1100 x 720; below it is `invalid_arguments`). A guest is refused the two sections with
  `unavailable`. Click `account.settings.tab.*` / `account.overlay.tab.*` only when the tab
  button itself is what you are proving; `get_buttons` shows their `intent`.
- An agent's screenshot is `launcher.view.capture` — `confirm` on EVERY call, local only:
  call it with `approve: card`, press `app_function.confirm.allow`, re-ask with the `call_id`;
  the PNG lands under `<support dir>/agent_attachments/launcher_captures/`. QA evidence
  captures stay `screenshot_window` (same renderer, no approval card).
- Library intents (one action layer S6): `launcher.library.select`, `.open_details`,
  `.close_details`, `.view_mode.set` (`showcase` / `rows` / `compact`), `.play` and `.update`
  (confirm), `.cancel`, beside `.list` / `.install` (confirm) / `.uninstall` (refused). A
  headless step ("be on item X with its details open") calls them with `call_app_function`;
  `click_button` stays for proving the button itself. Never `play` a real game in QA.
  `get_buttons` names each library control's intent, with the raw `item_id` from its `state`.
  Recipe: `stagec-library-select-vs-open-contract.md`.
- Posts intents (one action layer S8): `launcher.posts.feed.select` (`feed`: `for_you` /
  `following` / `explore`; opens Posts on it), `launcher.posts.feed.show_new` (the "N new
  posts" pill `posts.feed.pending`; refused when none are waiting) and `launcher.posts.interests.set` (confirm;
  booleans `cosplay` / `gaming` / `tech` / `news`, at least one true). Both need a signed-in
  account. `interests.set` writes the backend subscription first, so under `dev_login` it
  answers `failed` ("Session expired"); getting past the first-run prompt on such a session
  is still `dismiss_hashtag_onboarding` (and `open_app_tab`'s onboarding stage). The
  `posts.nav.<feed>` controls name `feed.select` as their intent; `click_button` on them
  stays the proof of the pills. The prompt's own controls are
  `blocking_modal.hashtag_onboarding.option.<tag>` (its chips) and `.save` (its Set hashtags,
  disabled until a chip is picked, naming `interests.set` with the picked tags); `.save`
  writes the backend, so under `dev_login` it fails and the prompt stays up.
- Settings and emulator setup (one action layer S7): `launcher.settings.get` / `.set` (confirm)
  carry `start_harness_with_launcher`; `launcher.emulators.list` (paired), `.add` / `.edit` /
  `.remove` (confirm; `kind` `emulator` or `folder`, ids from `list`), `.rescan`, and
  `.remove_with_files` (refused — the Launcher never deletes emulator or ROM files). Seed a
  QA emulator with `call_app_function` rather than driving the profile sheet; click
  `settings.emulators.*` only to prove the sheet or button. Point entries only at programs and
  folders the user already has — never fetch or name ROMs, BIOS, firmware or keys.
- Sounds: `launcher.audio.preview_cue` (`cue` `sent` / `arrived` / `click` / `ping`;
  allowed, local) plays one cue through Settings → Audio's preview door, refusing
  `unavailable` with the reason when the settings mute it. The
  `settings.audio.preview.<cue>` buttons name it as their intent.
- The AI page's agent directory: `launcher.agents.list` (paired read; `refresh: true`
  discovers again first) lists the discovered profiles' id, name, profile, harness,
  provider, model and `availability`, never an endpoint. `intelligence.agents.refresh`
  (Refresh discovery) names it; the page's other controls name none.
- Voice intents (one action layer S11), all local only: `launcher.voice.leave` (confirm),
  `launcher.voice.invite.accept` / `.decline` (confirm; the oldest ringing invite, the one
  `voice.state` lists first and the toast shows), `launcher.voice.share.sources` (`screen_0`,
  `window_2` — generic names, never a window title), `launcher.voice.share.start` (confirm;
  `source` from `sources`, optional `quality` `sharpness` / `smoothness`) and `.share.stop`.
  Each refuses `unavailable` with the reason when there is no call, no invite, or no share
  allowed. The `voice.call` / `voice.huddle` / `voice.share.start.<source>` / `.stop` controls
  name these as their intents; `click_button` on them stays the proof of the buttons. A real
  answer needs a second signed-in Launcher calling this one.
- Communities (one action layer S14): `launcher.communities.list` (paired; `which` `joined` or
  `discover`, optional `query` / `topic`), `.open` (`community` id or handle), `.create` and
  `.settings.set` (confirm; the wizard's and the pane's own rules refuse a bad value), `.join` /
  `.leave` (confirm; an owner's leave or an invite-only join is refused with the reason), and
  `.moderate` (refused — moderation acts on other people and stays the user's). The drawer tabs,
  topic chips, rows, cards, the detail's Join / Leave, the wizard's Create and the pane's Save
  name these as their intents;
  click them only to prove the surface. Under `dev_login` every one of these is backend-backed,
  so "Session expired" is the expected answer, not a defect. There is no moderation-queue entry:
  the queue is not built yet (communities wave 5).
- Mission Control (one action layer S12): `launcher.mission_control.panel.open` (`panel`
  `agents` / `console` / `telemetry` / `settings` / `board`, or `activity` for the command
  bar's Activity list through its chip; opens the tab first; with `telemetry`, an optional
  `tab` `runs` / `terminal` / `health`) and
  `.panel.close`, `.agent.select` (`agent_id`; an unknown id is refused with the roster's ids)
  and `.command_bar.pin` (`pinned`), all local only. Installs: `launcher.harness.installs.list`
  and `.reachable` (paired reads), `.select` / `.pair` (`payload`) / `.forget` (`install_id`,
  never this machine) and `.unpair` (`device_id`) — all confirm. Never pair, forget or unpair
  for real in QA: read the card, then Deny. Realm sync: `launcher.realm_sync.status` (paired)
  and `.open` (the Sync chip's sheet; refused while no realm is on screen). The drawer, agent,
  pin, install and `realm_sync.open` controls name these; the Telemetry tab aliases
  (`mission_control.drawer.runs` / `.terminal` / `.runtime`) name `panel.open` `telemetry`
  with `tab` `runs` / `terminal` / `health`; the Activity chip names `panel.open` `activity`
  while it is closed.
- Agent chat (one action layer S13): `launcher.agent_chat.list` (paired; the selected agent's
  saved chats, the open one, and the file names of the images the open chat shows), `.open`
  (`session_id` from `list`), `.new`, `.send` (confirm; `text`, sent through the console's own
  send and marked `app_function`, never as typed — refuses when no console is on screen),
  `.open_tools` / `.close_tools`, and `.attachment.open` / `.reveal` (confirm) / `.copy_path`
  (confirm) / `.open_in_studio` (Studio's agent gate) / `.close` — `file` is a name the chat
  shows or its full path, never an arbitrary path. Chat History's cards, `new_chat`,
  `open_newer`, `session_tools.close` and the image controls name these as their intents;
  `click_button` on them stays the proof of the buttons. `mission_control.agent_chat.inspect_tools`
  names `.open_tools` or `.close_tools` by its state's `open`; `mission_control.agent_chat.send`
  names none (the composer's text never crosses the envelope).
- A control's `intent` can depend on its state: the library deck's and hero's primary
  button name `launcher.library.play` / `.install` / `.update` / `.uninstall` by their
  state's `action`, and an emulator or folder switch names `launcher.emulators.edit` with
  the `id` and the `enabled` value its press sets. Read `intent` from `get_buttons` at the
  moment you act; it changes when the state does.
- Edit World (one action layer S15), local only, with the office mounted:
  `launcher.world.level.save_as` (confirm; `name`; only a level that follows a built-in, while
  Edit World is open), `.rename` (confirm; `name`; the user's own level, named or not), `.retry_load` (re-reads a
  level that failed to load and says whether it opened) and `.reset_unreadable` (refused — it
  replaces stored work; Reset stays the user's). Each refuses `unavailable` with the panel's
  reason where the panel shows no button. Retry and Reset name these as their intents; Save as
  new level and Name this level open a name dialog and name none. QA names go to a scratch
  level only; the canvas and the editor tools stay `pointer_drag` / `click_button`. No control
  or entry opens Edit World yet (mission-control-queue row), so on a fresh office every one of
  them answers `unavailable`, "Edit World is not open on the office."
- Education intents (one action layer S9/S10): `launcher.education.open_section`
  (`section` as in `education.nav.<section>`), `.explore.list` / `.library.list` (paired
  reads: Explore keys, library course ids, `can_open` / `can_resume`), `.course.open` /
  `.course.resume` (`course_id`), and — on the course OPEN in Education (`course_id`
  optional, must match) — `.course.start`, `.course.open_section` (`section_id`),
  `.lesson.open` (`lesson_id`); on the open lesson `.lesson.play_pause`, `.lesson.seek_cue`
  (`cue`, a built transcript row) and `.lesson.source_toggle`. Writes: `.course.install`
  (`key`), `.course.update.apply` and `.course.open_source` are confirm, `.update.dismiss`
  allowed, `.course.remove` refused. A headless "be on lesson X" step calls them; clicks stay
  the proof of the buttons, and `get_buttons` names each control's intent. On a real staging
  session answer Deny to install / update cards.
- Studio (one action layer S16, `docs/studio/planned/agent-operable-studio.md`) — only in a
  build made with `--dart-define=ETERNIA_STUDIO_AGENT_CONTROL=true`; every other build answers
  `opt_in_off`. All local only. Reads: `launcher.studio.project.list`, `.state` (open project,
  `document_revision` / `saved_revision`, undo/redo, busy, playing, selection) and
  `.layer.list` (top first, the panel's names) — both `unavailable` until Studio is open; a
  read never boots it. Screen moves (allowed): `.project.open` (`id`), `.character.open`
  (`draft_id`; mints the draft's project once), `.view.fit`, `.playback.toggle`,
  `.layer.select` (`layer_id`). Writes (confirm — `approve: card`, press
  `app_function.confirm.allow`, re-ask with the `call_id`, or `approve: allow` / `deny`):
  `.edit.undo` / `.edit.redo`, `.project.create` (`name`, default `Untitled project`),
  `.project.save`, `.media.import` (`path` on this machine; joins the OPEN project). Agent
  edits are journaled with the turn that asked. A headless "be in project X with layer Y
  selected" step calls these; the `studio.*` controls name them as their intents and stay the
  proof of the buttons. QA writes go to a scratch project only.
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
- A headless scroll is the intent: `call_app_function {"method": "launcher.view.scroll",
  "args": {"target": "shell", "direction": "down"}}` (or `"to": "top"|"bottom"`); its answer
  carries `moved`, `offset_before/after` and `can_scroll_*`. It presses the same page-scroll
  door the controls press. Straight after opening a page, wait for
  `launcher.view.content_state` = `ready` first: measured live, `to: bottom` on a Library
  still laying out answered `moved: false, can_scroll_down: false`.
- The open agent chat's conversation is target `agent_chat`: `up` / `top` move toward older
  messages, `down` / `bottom` toward the latest; it refuses `unavailable` with no chat on
  screen. The `mission_control.agent_chat.feed.older|newer|oldest|latest` controls name it;
  the `feed.reveal.<anchor>` controls stay QA-only.
- `click_button(id="shell.scroll.down")` is the proof of the control itself; verify
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
closes. While the deck is open, `get_buttons(scope='library.action')` also lists the deck's
action row: `library.action.deck.primary` (the big button) and `library.action.deck.<intent>`
for each secondary action (`uninstall`, `remove` — the emulator HIDE —, `repair`, …), each
`enabled` exactly when the button is. A press goes through the same dispatch as a click, so a
destructive one raises its confirmation dialog. In Rows view the hero banner's Play pill is
`library.action.hero.primary` (disabled while the deck is open). `get_widget_state(widget='library.store')`
reads the store's `install_record_count` / `install_binding_count`. Never treat an item selector as an opener, and never claim details opened from a
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

### The `/qa/callAppFunction` REST fallback

If the in-chat MCP wrapper schema rejects a call but a live QA marionette session already
exposes the redaction-safe port/nonce from its launch artifact, the bounded direct QA REST
route is an acceptable navigation fallback (the `/qa/setTab` route is gone with the verb):

- `POST http://127.0.0.1:<port>/qa/callAppFunction`
- header `X-Stagec-Qa-Nonce: <nonce>`
- body `{ "method": "launcher.navigation.open", "args": { "page": "missionControl" } }`

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

### Settle before capture, and `content_state`

All three capture tools first wait — capped by `settle_timeout_ms`, default 15 s — for
the app to say the selected tab's content has finished (the `getContentState` verb), so
a frame of the feed still loading is not taken by accident. `settle: "none"` captures at
once (use it for deliberate loading-state or motion frames).

Every capture result carries `content_state`, read from the app right after the frame:

- `ready` — the tab's content is on screen. Only this is proof of loaded content.
- `loading` — still loading when the frame was taken (the cap ran out:
  `settle.timed_out=true`). Report it as a loading frame, never as the content.
- `empty` — the tab finished with nothing to show. Honest, but say so.
- `error` — the tab is showing an error state. Report the error, not the screen.
- `unknown` — the app could not answer (a binary older than the verb). Do not present
  the frame as loaded content.

`content.source` says which signal answered: `tab_signal` (the tab's own typed states —
News rows / loading / empty / error, the Posts feed surface) or `progress_scan` (no tab
signal: an indeterminate spinner or skeleton on screen means `loading`). A non-ready
capture also carries `content_warning` (and a warning on `open_app_tab`).

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
2. Confirm the signed-in session if the surface needs one (section 0).
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

**Video is opt-in. Record only when a row or the operator explicitly asks for video.** A
screenshot stays the default proof. A recording is many full-window PNGs, and every frame
you open costs about as many tokens as a screenshot — so a 3 s recording at 4 fps is twelve
screenshots' worth if you read it all. Never record "just in case", and never record
because a screenshot would do.

When video is asked for, call `mcp_launcher_qa_record_video`. It is the in-app recorder: a
bounded burst of the app's own rendered frames (`captureFrame`, the same path as
`screenshot_window`), so the window is never raised, focused or resized, and nothing on the
desktop is recorded. Options and hard caps:

- `duration_ms` — default 3000, cap 200–15000.
- `fps` — default 4, cap 1–10. A slow render drops ticks (reported as `dropped_ticks`) rather
  than stretching time; read `achieved_fps`.
- `label` (filename-safe) and `out_dir` (default: the session artifact dir's `videos/`).

Anything outside a cap is refused (`invalid_request`), never clamped. The result carries
`recording_dir`, `manifest_path`, `frame_count`, `first_frame_path` and `last_frame_path` —
never frame bytes. `manifest.json` lists every frame's path, `t_ms` and size. No encoder
ships yet, so `encoded_video_path` is null and `encode_status` is `frames_only`; deliver
the recording directory, and open only the frames that show the claim.

The recorder captures what the app does on its own (animation, loading, playback,
transitions) while it runs; it does not drive the app meanwhile, and it holds the fixture
mutex, so set the state up first, then record. Refusals are named: `app_not_attached`,
`capture_in_app_unsupported` (rebuild the QA binary), `capture_frame_failed` (a partial
manifest is kept), `capture_corrupt_png`.

Do not record the desktop, and do not raise the window to make a recording work — a
window-title FFmpeg grab records whatever covers that region, which the ruling forbids.

**Stale artifacts are not proof.** An MP4 or PNG captured before the current fix/state is
invalid proof even if it is valid media. Re-capture after the change; never defend or
resend the old file. See `references/mission-control-current-video-proof-and-env-pinning.md`.

**Artifacts must show the claimed state.** If a live chat turn, board change, or roster
change drove the claim, the pixels must show the corresponding panel state. CLI/Harness
success without matching visible evidence is not UI proof — report the Launcher/read-model
mismatch as a high-severity regression instead of presenting the artifact as success.

---

## 7. Credential preflight and recovery

A real signed-in session is the persisted `stagec-smoke` session (section 0); `dev_login`
is chrome only. The staging credential helper is the other real lane:
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
  stale, the redaction-safe QA control REST `/qa/callAppFunction` path; deliver the PNG immediately
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
