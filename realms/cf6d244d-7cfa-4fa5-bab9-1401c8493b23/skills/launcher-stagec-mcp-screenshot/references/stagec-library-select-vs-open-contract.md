# Stage C Library MCP: select vs open contract

Session lesson: Library item semantic controls must be verb-explicit because `click_button` can target both selectors and open/details actions.

## Current contract

- Library item controls are selectors, not openers:
  - id shape: `library.item.select.<safe-item-id>`
  - scope: `library.item`
  - kind: `select`
  - label shape: `Select <title>`
- Opening the selected item/details remains a separate action:
  - id: `library.action.open_details`
  - scope: `library.action`
  - kind: `primary`
- Close action:
  - id: `library.action.close_details`

## Correct runner sequence

1. Call `get_buttons(scope='library.item')` and choose an id that starts with `library.item.select.`.
2. Call `click_button(id='<library.item.select...>')` to focus/select the Library item.
3. Verify selection via either:
   - `get_buttons(scope='library.item')` selected=true on that id, or
   - click response `library_focus_changed=true` / `library_focus_after=<item-id>`.
4. Call `click_button(id='library.action.open_details')` to open the details/post panel.
5. Verify open state with `get_widget_state(widget='library.details')`:
   - `details_open=true`
   - `selected_matches_details=true`
   - `details_item_id_safe` equals selected id.

## Pitfall

Do not treat `library.item.*` as an open/details click. If a runner sees the old ambiguous id shape (`library.item.<safe>` without `.select.`), consider the app/MCP binary stale and rebuild/relaunch before claiming the contract is current.

## Verification pattern

- Unit/contract gates used in the session:
  - `flutter analyze ...stagec/library QA files...`
  - `flutter test test/core/qa/library_qa_observatory_test.dart test/core/qa/stagec_qa_command_bus_stage19_buttons_test.dart`
  - from `tool/stagec_qa_mcp_server`: `dart test test/buttons_tools_test.dart test/schema_validate_test.dart`
- Live smoke:
  - rebuild `flutter build windows --debug -t lib/main_marionette.dart`
  - open Library via MCP
  - `get_buttons(scope='library.item')` should show `library.item.select.*`, `kind: select`, and labels prefixed with `Select `.
