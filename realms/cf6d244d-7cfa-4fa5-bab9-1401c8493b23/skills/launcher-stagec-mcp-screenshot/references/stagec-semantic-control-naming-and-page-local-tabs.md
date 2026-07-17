# Stage C semantic control naming and page-local tabs

Session date: 2026-05-18

## Trigger

During live Library MCP exploration, `click_button(id="library.item.external:steam:2074920")` selected a Library item but did not open details. The user correctly flagged that the first click was ambiguous: item controls looked like open/click controls while their behavior was selector-only.

## Durable lesson

Semantic MCP IDs must encode the user-visible verb when a visible control has multiple plausible meanings. For Library:

- Selector-only item controls should use:
  - id: `library.item.select.<safe-id>`
  - label: `Select <title>`
  - kind: `select`
- Opening details should remain separate:
  - id: `library.action.open_details`
  - label: `Open details for selected item`
  - kind: `primary`

Do not normalize ambiguous IDs like `library.item.<safe-id>` in agent workflow. Treat them as a contract defect and fix/escalate.

## Correct Library MCP workflow

1. Enumerate selectors:
   - `get_buttons(scope="library.item")`
2. Select the item:
   - `click_button(id="library.item.select.external:steam:2074920")`
3. Open details explicitly:
   - `click_button(id="library.action.open_details")`
4. Verify semantic state:
   - `get_widget_state(widget="library.details")`
   - Expect `details_open=true`, matching `details_item_id_safe` and `selected_item_id_safe`.
5. Optionally capture screenshot proof with `screenshot_window`.

## Regression probe

After rebuilding/relaunching the Marionette EXE, verify:

- Old ambiguous id fails closed:
  - `click_button(id="library.item.external:steam:2074920")` -> `button_not_found`
- New explicit id succeeds:
  - `click_button(id="library.item.select.external:steam:2074920")` -> `ok=true`, `kind="select"`, `library_focus_changed=true`

## Education page-local tabs gap

On the Education tab, the UI visibly had page-local subtabs:

- Library
- Explore
- Studio
- Tutor
- Passport
- Enterprise
- Demand
- Power user

But `get_buttons()` exposed only `shell.nav` controls. Attempts to `click_button(label="Explore")` failed with `button_not_found`. This is not an agent navigation mistake; it is a missing semantic-control surface.

Recommended scope shape:

- `education.nav.library`
- `education.nav.explore`
- `education.nav.studio`
- `education.nav.tutor`
- `education.nav.passport`
- `education.nav.enterprise`
- `education.nav.demand`
- `education.nav.power_user`

Until that exists, do not coordinate-click Education subtabs when the user asks for MCP commands. Report the semantic gap and continue only with exposed semantic controls.

## Related pitfall

`wait_for_state` path names are stricter than some compact widget envelopes. If an assertion path is missing but a direct state read (for example `get_navigation_state`) shows the truth, report the path mismatch as a QA/tooling issue rather than treating the target page as not navigated.
