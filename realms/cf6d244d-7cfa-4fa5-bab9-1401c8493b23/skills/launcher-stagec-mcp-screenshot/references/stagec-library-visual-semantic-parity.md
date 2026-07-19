# Stage C Library visual/semantic parity mismatch

Use this when validating Launcher Library MCP item selection, carousel focus, scroll, or details-open behavior.

## Trigger evidence

A direct Alice MCP session opened Library and successfully called `click_button` on several `library.item.*` controls. `get_buttons(scope=library.item)` later reported `selected=true` for `The First Descendant` (`library.item.external:steam:2074920`), but the real PrintWindow screenshot still visually showed the focused/centered Library card as `OVR Toolkit`.

Observed scroll mismatch from the same session:

- `mcp_launcher_qa_scroll(target="shell", direction="down", amount="page")` failed with `Target "shell" is not mounted; scroll cannot run yet`.
- `mcp_launcher_qa_get_widget_state(widget="shell")` simultaneously reported `mounted=true`, `selected_tab=library`.
- Current scroll schema lacked a dedicated Library target such as `library.items`, `library.grid`, or `library.carousel`.

## Rule

When semantic MCP state and screenshot evidence disagree, screenshot/visible UI wins for user-facing claims. Treat the disagreement as a parity bug and route audit/fix work; do not claim the item visibly flipped or selection passed based only on `click_button(ok=true)` or `get_buttons(... selected=true)`.

## QA acceptance pattern

For Library item selection/scroll claims, require all of:

1. Launch/open Library through MCP, with authenticated app/window/qa readiness proven.
2. Enumerate Library items with `get_buttons(scope=library.item)`.
3. Click/select the intended item with `click_button` by stable `library.item.*` id.
4. Read back semantic state with `get_buttons` or a widget state hook.
5. Capture a real PrintWindow screenshot and visually confirm the focused/centered card changed to the intended item.
6. If opening details is part of the claim, explicitly click `library.action.open_details` and verify `library.details` mounted/details-open state; do not equate selection with details-open.
7. Redaction-scan artifacts before reporting PASS.

## Fix/audit guidance

Audit should identify the exact source files/controllers for:

- visible carousel/card focus,
- semantic `library.item.selected`,
- details/open target selection,
- scroll target registration and mapping.

Likely implementation shape is a small parity fix: make semantic item controls drive the same controller/state path as visible focus, add a real Library scroll target abstraction, and add regression evidence that combines semantic state plus screenshot proof.
