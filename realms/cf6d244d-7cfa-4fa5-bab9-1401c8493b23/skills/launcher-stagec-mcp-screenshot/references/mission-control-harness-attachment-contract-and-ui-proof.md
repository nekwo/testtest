# Mission Control Harness attachment contract + UI proof lesson

Session lesson from a Mission Control/Harness cockpit upgrade pass:

## Attachment-capable Harness messaging

Harness agent chat should not forbid attachments permanently. Images/screenshots and short videos are valid operator context/proof inputs when they are passed as structured, redaction-aware handles rather than raw filesystem access.

Canonical first-pass attachment shape:

```json
{
  "kind": "image",
  "mime": "image/png",
  "uri": "artifact://mission-control/screenshot.png",
  "safe_label": "Mission Control screenshot"
}
```

First-pass capability allowlist:

- `persona.message_task`
- `persona.instance.message`
- `persona.instance.run_once`
- `persona.diagnose`

Do not add `attachments` to worker/run/daemon/archive control capabilities. Validate attachment args as a list of objects with `kind`, `mime`, `uri`, and optional string `safe_label`; allowed `kind` values are `image`, `video`, and `file`.

## Launcher UI/test implementation patterns

When adapting this contract in Launcher Mission Control:

1. Mirror `attachments` in the Launcher `HarnessCapabilitySpec` registry.
2. Add structured validation in `HarnessCapabilitySpec.validate` instead of accepting arbitrary extra args.
3. Add tests proving attachments work for persona message/run capabilities and fail for malformed entries/unrelated controls.
4. For Mission Control/office tests with persistent animation/tickers, avoid `pumpAndSettle()` in drawer helpers; use bounded pumps for known transitions.
5. If `scrollUntilVisible` receives a keyed `SingleChildScrollView`, target the descendant `Scrollable` and select a single candidate with `.first`.
6. Use semantic row keys for agent selection (for example `agent_log_picker_qa`) rather than tapping ambiguous visible text like `QA Agent`.
7. Keep role/agent labels visually separate from mission/task context in agent log rows so the UI is readable and automation can target the role name cleanly.
8. Expand the first terminal/proof event by default when it carries proof/decision metadata so fields like `Next expected` are immediately visible.
9. Always render a `Thinking / process summary` card for a selected role, including no-event/idle roles; otherwise the Harness feels dead even when the role is present.

## Visual proof interpretation

A screenshot can be valid proof that the page mounted and is non-blank while still proving the design is not AAA yet. Do not overclaim. If the Mission Control screenshot still shows mostly empty black canvas, primitive node cards, duplicate agent labels, or weak “computing” affordances, report those as remaining product/UI gaps even when deterministic tests pass.

## Verification bundle used in this class of change

- Hermes: `python -m pytest -q tests/agent_runtime/test_capabilities.py`
- Launcher: `flutter test test/features/mission_control/harness_capability_registry_test.dart`
- Launcher page: `flutter test test/features/mission_control/mission_control_page_test.dart`
- If MCP reports `launch_wrong_debug_target_missing_marionette`, rebuild with `flutter build windows --debug --target lib/main_marionette.dart`.
- If rebuild fails on locked `WebView2Loader.dll`, close/kill the stale Launcher process and rerun the build; capture the fix as lock recovery, not a product defect.
