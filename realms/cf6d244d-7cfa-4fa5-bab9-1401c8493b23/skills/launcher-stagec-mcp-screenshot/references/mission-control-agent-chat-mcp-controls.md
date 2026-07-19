# Mission Control Agent Chat MCP Controls

Session lesson: Launcher Mission Control can appear visually reachable through Stage C MCP while still not being interactively testable. A screenshot of the Agent Console is not enough; the selected Operator Channel needs stable semantic controls so MCP can click the actual chat/actions without coordinate fallbacks.

## Durable pattern

1. Launch Mission Control through `mcp_launcher_qa_open_app_tab` with explicit pins:
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
3. Open the Agent Console via semantic control:
   - `mission_control.drawer.agents`
4. Require unfiltered `get_buttons()` to expose selected Operator Channel controls such as:
   - `mission_control.agent_chat.send_test_message`
   - `mission_control.agent_chat.action.test_persona`
   - `mission_control.agent_chat.action.run_one_turn`
   - `mission_control.agent_chat.action.nudge`
   - `mission_control.agent_chat.action.pause`
   - `mission_control.agent_chat.action.resume`
   - `mission_control.agent_chat.action.cancel_run`
   - `mission_control.agent_chat.action.approve_run`
5. Click by stable `id`, not coordinates. Use `get_buttons()` readback and a screenshot after the click.

## If controls appear in app but get dropped

If `get_buttons()` reports `registry_dropped_reasons` with `unknown_scope` for Mission Control controls, update all three layers together:

- App-side QA observatory / marionette hook emits the scope and controls.
- In-app `StageCQaCommandBus` known scope/kind allowlists include the new scope/kind.
- Stage C MCP server tool schema enum includes the new scope.

Do not accept a partial fix where unfiltered controls are generated but the bus drops them.

## Stale tool-schema caveat

After patching the MCP server schema, the current chat/runtime may still have an already-loaded tool schema that rejects a new `scope` value before dispatch. In that case:

- unfiltered `get_buttons()` can still prove the live app emits the controls;
- `click_button(id=...)` can still exercise the control because id-clicks do not need the new scope enum;
- a Hermes/MCP reload is required before scoped queries like `get_buttons(scope=mission_control.agent_chat)` work in the chat tool schema.

Report this as a schema reload/freshness issue, not as product behavior proof.

## Claude-grade acceptance rule

For Mission Control agent chat, successful MCP mechanics are not product success. After `mission_control.agent_chat.send_test_message`, the UI must show one of:

- assistant response in the same transcript;
- live queued/running/tool/completed turn state with bounded refresh;
- precise actionable blocker.

A state that only shows Harness accepted/queued, with no automatic turn/response, remains a runtime/product gap even if the MCP click succeeds.

## Build freshness pitfall

If rebuilding `lib/main_marionette.dart` fails because `WebView2Loader.dll` is locked by `eternia_launcher`, close/kill the attached Launcher first via MCP, then rebuild. This is a normal freshness step before live MCP proof, not a product defect.
