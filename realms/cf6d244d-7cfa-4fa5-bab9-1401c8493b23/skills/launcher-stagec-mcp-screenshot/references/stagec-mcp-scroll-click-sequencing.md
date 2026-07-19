# Stage C MCP scroll/click sequencing: schema first, verify every verb

## Lesson

Do not assume every Launcher surface has a scroll target or that a visible item click opens details. The MCP schema is the contract.

A recent Library workflow mistake came from calling the generic `scroll` tool against Library even though the tool schema only supported `shell`, `news.feed`, and `posts.feed`, then treating the first item click as if it might open a post/details page. In Library, item controls are selector/focus actions unless a separate action such as `library.action.open_details` is exposed.

## Correct sequence

1. Open/navigate with `open_app_tab` / `set_tab` and verify `navigation.selected_tab`.
2. Enumerate current semantic controls with `get_buttons()` scoped to the relevant surface.
3. Before using `scroll`, check the tool schema target list. If the desired surface is not supported, do not call it; record a missing MCP scroll-surface gap if scrolling is required.
4. For Library:
   - click `library.item.select.<safe-id>` or scoped `library.item` selector to select/focus the item,
   - verify selected state if available,
   - click `library.action.open_details` / explicit details action,
   - verify `get_widget_state(widget=library.details)` has `details_open=true` and `selected_matches_details=true`,
   - capture screenshot evidence only after state verification.
5. User-facing claims should name the actual sequence: selected item first, opened details second.

## Pitfall

If a tool call returns `command_unavailable` because the target is not in the schema, the agent should not frame that as a mysterious app failure. It is an affordance/schema mismatch. Escalate a semantic MCP gap only if the workflow truly needs that target wired.
