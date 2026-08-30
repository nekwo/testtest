# Library Semantic MCP Click and Scroll Pitfalls

Use this when driving the Eternia Launcher Library tab through Stage C MCP semantic controls.

## Durable lesson

Do not assume generic scroll/click semantics apply to Library. The Stage C MCP scroll tool may only support specific targets such as `shell`, `news.feed`, and `posts.feed`; if Library is not listed in the tool schema, do not call scroll against a fictional Library target.

For Library cards/items, a semantic item click may only **select** the item. Opening details can require a separate explicit action such as `Open details for selected item` / a registered details-action control.

## Correct sequence

1. Open/navigate to Library with the MCP composer or shell tab control.
2. Enumerate registered semantic controls with `get_buttons`, scoped to Library when possible.
3. Treat Library item controls as selection unless their id/label explicitly says open/details/play.
4. Click the item/select control.
5. Verify state after selection, for example `library.details` / selected item fields.
6. Click the explicit details/open action if present.
7. Verify `details_open`, `details_item_id_safe`, and `selected_matches_details` before reporting that a post/item/details page is open.
8. Capture screenshot evidence only after state verification.

## Reporting discipline

If the first click only selected an item, say that plainly. Do not describe the item-click as “opened” unless the details state proves it. When a tool schema makes a target unsupported, call it an MCP affordance gap and switch to registered controls rather than trying unsupported scroll targets.
