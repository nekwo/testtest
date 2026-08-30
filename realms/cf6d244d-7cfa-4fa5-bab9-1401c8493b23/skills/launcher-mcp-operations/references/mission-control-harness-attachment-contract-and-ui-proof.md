# Harness attachment handle contract and honest UI proof

> **Refresh note (2026-08-28):** the `persona.message_task` capability and the
> stage/proof-metadata expansion guidance belonged to the removed goal/task mission lane
> and are dropped. The attachment-handle shape, the Launcher test patterns, and the
> honest-visual-proof rule are the durable content.

## Attachment-capable Harness messaging

Harness agent chat should not forbid attachments permanently. Images/screenshots and short
videos are valid operator context/proof inputs when they are passed as structured,
redaction-aware handles rather than raw filesystem access.

Canonical attachment shape:

```json
{
  "kind": "image",
  "mime": "image/png",
  "uri": "artifact://launcher/screenshot.png",
  "safe_label": "Launcher screenshot"
}
```

Capability allowlist for attachments:

- `persona.instance.message`
- `persona.instance.run_once`
- `persona.diagnose`

Validate attachment args as a list of objects with `kind`, `mime`, `uri`, and an optional
string `safe_label`; allowed `kind` values are `image`, `video`, and `file`. Do not attach
this arg to unrelated control capabilities.

## Launcher UI/test implementation patterns

When adapting this contract in the Launcher cockpit:

1. Mirror `attachments` in the Launcher `HarnessCapabilitySpec` registry.
2. Add structured validation in `HarnessCapabilitySpec.validate` instead of accepting
   arbitrary extra args.
3. Add tests proving attachments work for persona message/run capabilities and fail for
   malformed entries and unrelated controls.
4. For cockpit/office tests with persistent animation/tickers, avoid `pumpAndSettle()` in
   drawer helpers; use bounded pumps for known transitions.
5. If `scrollUntilVisible` receives a keyed `SingleChildScrollView`, target the descendant
   `Scrollable` and select a single candidate with `.first`.
6. Use semantic row keys for agent selection (for example `agent_log_picker_qa`) rather
   than tapping ambiguous visible text like `QA Agent`.
7. Keep role/agent labels visually separate from surrounding context in agent log rows so
   the UI is readable and automation can target the role name cleanly.
8. Always render a `Thinking / process summary` card for a selected role, including
   no-event/idle roles; otherwise the cockpit feels dead even when the role is present.

## Visual proof interpretation

A screenshot can be valid proof that the page mounted and is non-blank while still proving
the design is not AAA yet. Do not overclaim. If the screenshot shows mostly empty canvas,
primitive node cards, duplicate agent labels, or weak affordances, report those as
remaining product/UI gaps even when deterministic tests pass.

## Verification bundle used in this class of change

- Hermes: `python -m pytest -q tests/agent_runtime/test_capabilities.py`
- Launcher: `flutter test test/features/mission_control/harness_capability_registry_test.dart`
- Launcher page: `flutter test test/features/mission_control/mission_control_page_test.dart`
- If MCP reports `launch_wrong_debug_target_missing_marionette`, rebuild with
  `flutter build windows --debug --target lib/main_marionette.dart`.
- If the rebuild fails on a locked `WebView2Loader.dll`, close/kill the stale Launcher
  process and rerun the build; capture that as lock recovery, not a product defect.
