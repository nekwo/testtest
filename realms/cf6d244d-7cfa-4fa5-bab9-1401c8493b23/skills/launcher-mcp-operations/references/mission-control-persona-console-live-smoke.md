# Persona console live smoke

> **Refresh note (2026-08-28):** the run-gated chat actions (`run_one_turn`, `cancel_run`,
> `approve_run`) were removed with the goal/task mission lane and are dropped from the
> expected control set; everything else is unchanged.

Session lesson: when the operator asks whether he can talk to Neko or another persona,
verify the Agent Console with semantic MCP controls, not just a screenshot or the Launcher
navigation state.

## Durable pattern

1. Launch/open the cockpit through Stage C MCP with smoke credentials and explicit env
   pins:
   - `credential_profile: stagec-smoke`
   - `browser_login: true`
   - `hermes_profile: alice` or the intended profile
   - `harness_runtime_root` set to the intended Agent Runtime Harness root
   - `hermes_home` set to the intended Hermes profile home
2. If MCP fails with `launch_wrong_debug_target_missing_marionette`, rebuild the Windows
   debug EXE with the Marionette target, then relaunch:
   - `flutter build windows --debug --target lib/main_marionette.dart`
   - Use the `/FS` compiler flag if MSVC PDB contention is possible.
3. Verify the launch envelope before interacting:
   - `ok=true`
   - `auth_state.status=authenticated`
   - `navigation_state.selected_tab=missionControl`
   - `app.qa_control_ready=true`
   - env pin fields are configured/non-null.
4. Enumerate controls with `get_buttons(include_disabled=true, include_invisible=false)`.
5. Open the Agent Console semantically: click `mission_control.drawer.agents`.
6. Verify persona selection controls are present and selectable, for example:
   - `mission_control.agent.select.personainst_neko_supervisor`
   - `mission_control.agent.select.personainst_dev`
   - `mission_control.agent.select.personainst_backend_dev`
   - `mission_control.agent.select.personainst_qa`
7. Verify operator-message controls in `mission_control.agent_chat`:
   - `mission_control.agent_chat.send_test_message` should be enabled for the selected
     supported persona.
   - `mission_control.agent_chat.action.test_persona` should be enabled for
     sandbox/persona smoke.
8. For a quick smoke, click `mission_control.agent_chat.send_test_message` after selecting
   Neko or another persona and require MCP `ok=true` for the click.
9. Capture a PNG after opening the Agent Console for operator-facing proof.

## Reporting boundary

This proves the persona selector and the prewired test-message action are live. It does
**not** by itself prove a freeform typed custom message round trip, or that a new agent
response event appears in the transcript. If that deeper loop was not tested, say so
explicitly and name it as the remaining AAA proof lane.

## Pitfalls

- Do not treat navigation/auth alone as proof that the operator can talk to personas; the
  `mission_control.agent` and `mission_control.agent_chat` controls must be visible and
  enabled.
- Do not overclaim a disabled state-dependent action as broken. Enumerate the live
  registry and read each control's `enabled` state before judging it.
- If the EXE was overwritten by a normal `main.dart` build, the right fix is a Marionette
  rebuild, not a product bug claim.
