# Mission Control persona console live smoke

Session lesson: when Tony asks whether he can talk to Neko / each persona, verify the Mission Control Agent Console with semantic MCP controls, not just a screenshot or Launcher navigation state.

## Durable pattern

1. Launch/open Mission Control through Stage C MCP with smoke credentials and explicit env pins:
   - `credential_profile: stagec-smoke`
   - `browser_login: true`
   - `hermes_profile: alice` or the intended profile
   - `harness_runtime_root` set to the intended Agent Runtime Harness root
   - `hermes_home` set to the intended Hermes profile home
2. If MCP fails with `launch_wrong_debug_target_missing_marionette`, rebuild the Windows debug EXE with the Marionette target, then relaunch:
   - `flutter build windows --debug --target lib/main_marionette.dart`
   - Use the `/FS` compiler flag if MSVC PDB contention is possible.
3. Verify the launch envelope before interacting:
   - `ok=true`
   - `auth_state.status=authenticated`
   - `navigation_state.selected_tab=missionControl`
   - `app.qa_control_ready=true`
   - env pin fields are configured/non-null.
4. Enumerate controls with `get_buttons(include_disabled=true, include_invisible=false)`.
5. Open the Agent Console semantically:
   - click `mission_control.drawer.agents`
6. Verify persona selection controls are present and selectable, for example:
   - `mission_control.agent.select.personainst_neko_supervisor`
   - `mission_control.agent.select.personainst_dev`
   - `mission_control.agent.select.personainst_backend_dev`
   - `mission_control.agent.select.personainst_qa`
7. Verify operator-message controls in `mission_control.agent_chat`:
   - `mission_control.agent_chat.send_test_message` should be enabled for the selected supported persona.
   - `mission_control.agent_chat.action.test_persona` should be enabled for sandbox/persona smoke.
   - `run_one_turn`, `cancel_run`, and `approve_run` may be state-dependent and disabled when no runnable/active task exists.
8. For a quick smoke, click `mission_control.agent_chat.send_test_message` after selecting Neko or another persona and require MCP `ok=true` for the click.
9. Capture a PNG after opening the Agent Console for Tony-facing proof.

## Reporting boundary

This proves the Mission Control persona selector and prewired test-message action are live. It does **not** by itself prove a freeform typed custom message round trip or a new agent response event appears in the transcript. If that deeper loop was not tested, say so explicitly and name it as the remaining AAA proof lane.

## Pitfalls

- Do not treat Mission Control navigation/auth alone as proof that Tony can talk to personas; the `mission_control.agent` and `mission_control.agent_chat` controls must be visible and enabled.
- Do not overclaim disabled state-dependent actions as broken. `Run One Turn`, `Cancel Run`, and `Approve Run` can be disabled when no matching active/runnable state exists.
- If the EXE was overwritten by a normal `main.dart` build, the right fix is a Marionette rebuild, not a product bug claim.
