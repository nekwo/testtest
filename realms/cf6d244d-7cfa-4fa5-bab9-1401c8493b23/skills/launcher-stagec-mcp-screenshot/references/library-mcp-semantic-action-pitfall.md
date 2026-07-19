# Library MCP Semantic Action Pitfall

Use when driving the Eternia Launcher Library tab with Stage C MCP semantic controls.

## Durable lesson

Do not assume a Library item click opens details. In the current semantic control model, item click may only select the row/card. Opening details can require an explicit action such as `Open details for selected item` / details action after selection.

## Correct interaction sequence

1. Enumerate semantic controls for the relevant Library scopes before acting.
2. Treat `library.item` clicks as selection unless the registered control id/label explicitly says open/details/play.
3. After item selection, use the explicit details/open action if present.
4. Verify state after each step, especially:
   - selected item id/label
   - `library.details.details_open`
   - selected item matches details item
5. Only claim the post/game/details page is open after the state oracle confirms it, not merely after the first item click.

## Scroll target schema discipline

Use MCP scroll only for targets supported by the tool schema. If `scroll` only lists `shell`, `news.feed`, and `posts.feed`, do not call it against Library. For Library, prefer semantic navigation/actions or a Library-specific registered control if one exists.

## Reporting

If a scroll target is unavailable, state that the tool target is not wired rather than treating it as product failure. Then proceed via supported semantic controls and report the exact state verification.
