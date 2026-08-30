# Mission Control drawer semantic controls and the Agent Console

> **Refresh note (2026-08-28):** the mission/evidence/runs drawers belonged to the removed
> goal/task mission lane and are dropped, along with the "projector tuning" follow-up (the
> projector is retired). The drawer semantic-control contract and the Agent Console proof
> path are the durable content.

## When this applies

Use this when Launcher cockpit work adds or verifies page-local drawers such as the Agent
Console, Live Terminal, or Harness Runtime drawer.

## Durable lesson

Drawer proof should use page-local semantic controls, not coordinate clicks or generic
shell navigation. A valid proof path is:

1. Launch/open Stage C to `tab=missionControl` with MCP.
2. Enumerate semantic buttons with `get_buttons(include_disabled=true,
   include_invisible=true)`.
3. Verify the `mission_control.drawer.*` IDs are present.
4. Click the exact drawer ID, e.g. `mission_control.drawer.agents`.
5. Re-read buttons and verify the clicked drawer has `selected=true` and
   `mission_control.drawer.close` becomes enabled.
6. Capture the current window with `screenshot_window` and visually sanity-check that the
   requested drawer is actually open.

Expected drawer IDs:

- `mission_control.drawer.agents` — opens the Agent Console / Harness Agents drawer.
- `mission_control.drawer.terminal` — opens the Live Terminal drawer.
- `mission_control.drawer.runtime` — opens the Harness Runtime drawer.
- `mission_control.drawer.close` — closes the active drawer.

Enumerate rather than assume: the live registry is authoritative, and drawer IDs are added
and removed as the cockpit changes. If a drawer you expect is absent from `get_buttons()`,
record the semantic-control gap; do not coordinate-click it.

## Implementation pattern

- Page state should register a QA-only observatory/controller from the mounted page.
- The Stage C marionette hooks should build `mission_control.drawer` controls only when
  `selected_tab == missionControl` and the page controller is mounted.
- `click_button(id=...)` should dispatch through the controller and then expose selected
  state via the next `get_buttons()` read.
- Do not add coordinate-only click surfaces.

## Pitfalls from the Agent Console migration

- Production `MessageFeed` / `MessageComposer` need a finite height. If embedded inside a
  `SingleChildScrollView` drawer, wrap the operator channel in a bounded
  `SizedBox`/layout boundary when the parent height is unbounded.
- The cockpit may wrap its content in `SelectionArea`; the production DM stack owns its
  own selectable/text-input internals. Disable the ancestor selection boundary only around
  the operator-channel DM stack to avoid Flutter selectable-region crashes during drawer
  transitions.
- `MessageComposer` is Riverpod-backed. Widget tests that mount it directly or through
  cockpit surfaces need a `ProviderScope`.
- Shared `SystemMessageTile` rows must wrap/ellipsis long system/Harness labels. Long
  uppercase system rows can otherwise overflow horizontally in operator-channel tests.

## Proof acceptance

A strong proof bundle includes:

- `open_app_tab(tab=missionControl)` with `ok=true`, `selected_tab=missionControl`,
  `auth_state.status=authenticated`, and `qa_control_ready=true`.
- `get_buttons()` showing `mission_control.drawer.agents` and sibling controls.
- `click_button(id="mission_control.drawer.agents")` returning `ok=true`.
- A follow-up `get_buttons()` showing `mission_control.drawer.agents selected=true`.
- A `screenshot_window()` PNG that visibly shows `Agent Console`, `Harness Agents`, and the
  operator-channel composer.

If the launch envelope has `harness_runtime_root_configured=false`, report the screenshot
as UI/control proof only, not pinned live Harness data-parity proof.
