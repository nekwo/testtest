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

## Headless runner sequence (the intent, not the button)

When the step is only a precondition — "be on item X, details open" — call the
app functions; they run through the Library's own doors (one action layer S6):

1. `call_app_function {"method": "launcher.library.list"}` and take an item `id`.
2. `call_app_function {"method": "launcher.library.view_mode.set", "args": {"mode": "showcase"}}`.
3. `call_app_function {"method": "launcher.library.select", "args": {"item_id": "<id>"}}`
   — opens the Library and focuses the item (sectioned showcase included); refused
   outside the showcase view.
4. `call_app_function {"method": "launcher.library.open_details", "args": {"item_id": "<id>"}}`,
   and `launcher.library.close_details` to close it.
5. Verify with `get_widget_state(widget='library.details')` as below.

`launcher.library.play` / `.update` are confirm entries (`approve`: `allow`, `deny`
or `card`); never play a real game in QA.

## Click sequence (proving the controls themselves)

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
  - from `tool/stagec_qa_mcp_server`: `dart test test/buttons_tools_library_test.dart test/buttons_tools_library_nav_test.dart test/schema_validate_library_enums_test.dart`
- Live smoke:
  - rebuild `flutter build windows --debug`
  - open Library via MCP
  - `get_buttons(scope='library.item')` should show `library.item.select.*`, `kind: select`, and labels prefixed with `Select `.
