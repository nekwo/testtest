# Agent Console chat MCP controls

> **Refresh note (2026-08-28):** the run-gated actions (`run_one_turn`, `approve_run`,
> `cancel_run`, `nudge`, `pause`, `resume`) belonged to the removed goal/task mission lane
> and are dropped from the expected control set. The three-layer allowlist rule, the
> stale-schema caveat, and the acceptance rule are the durable content.

Session lesson: the Launcher cockpit can appear visually reachable through Stage C MCP
while still not being interactively testable. A screenshot of the Agent Console is not
enough; the selected Operator Channel needs stable semantic controls so MCP can click the
actual chat actions without coordinate fallbacks.

## Durable pattern

1. Launch the cockpit through `mcp_launcher_qa_open_app_tab` with explicit pins:
   - `hermes_profile`
   - `harness_runtime_root`
   - `hermes_home`
   - `profile: stagec-smoke`
2. Verify attach/readiness before visual proof:
   - `ok=true`
   - `selected_tab=missionControl`
   - `auth_state.status=authenticated`
   - `qa_control_ready=true`
   - env pins configured in the launch envelope.
3. Open the Agent Console via semantic control: `mission_control.drawer.agents`.
4. Require unfiltered `get_buttons()` to expose the selected Operator Channel controls
   under scope `mission_control.agent_chat`, for example:
   - `mission_control.agent_chat.send_test_message`
   - `mission_control.agent_chat.action.test_persona`

   Enumerate rather than assume a fixed list — the live registry is authoritative.
5. Click by stable `id`, not coordinates. Use a `get_buttons()` readback and a screenshot
   after the click.

## If controls appear in the app but get dropped

If `get_buttons()` reports `registry_dropped_reasons` with `unknown_scope`, update all
three layers together:

- the app-side QA observatory / marionette hook that emits the scope and controls;
- the in-app `StageCQaCommandBus` known scope/kind allowlists;
- the Stage C MCP server tool schema enum.

Do not accept a partial fix where unfiltered controls are generated but the bus drops them.

## Stale tool-schema caveat

After patching the MCP server schema, the current chat/runtime may still hold an
already-loaded tool schema that rejects a new `scope` value before dispatch. In that case:

- unfiltered `get_buttons()` can still prove the live app emits the controls;
- `click_button(id=...)` still exercises the control, because id-clicks do not need the new
  scope enum;
- a Hermes/MCP reload is required before scoped queries like
  `get_buttons(scope=mission_control.agent_chat)` work in the chat tool schema.

Report this as a schema reload/freshness issue, not as product behavior proof.

## Claude-grade acceptance rule

Successful MCP mechanics are not product success. After
`mission_control.agent_chat.send_test_message`, the UI must show one of:

- an assistant response in the same transcript;
- a live queued/running/tool/completed turn state with bounded refresh;
- a precise, actionable blocker.

A state that only shows Harness accepted/queued, with no automatic turn or response,
remains a runtime/product gap even if the MCP click succeeded.

## Build freshness pitfall

If rebuilding `lib/main_marionette.dart` fails because `WebView2Loader.dll` is locked by
`eternia_launcher`, close/kill the attached QA Launcher first, then rebuild. This is a
normal freshness step before live MCP proof, not a product defect. Never kill the
operator's own live Launcher to unblock a build without asking — record the exact PID and
ask.
