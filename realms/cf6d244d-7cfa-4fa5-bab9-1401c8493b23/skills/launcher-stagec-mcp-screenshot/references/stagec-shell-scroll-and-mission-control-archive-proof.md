# Stage C shell scroll + Mission Control archive proof

## Lesson

Mission Control can have the relevant proof surface below the first viewport: `Ready for Archive`, `Mission Playback`, `Archive Controls`, and the `View Archive` / archived-history state may not all be visible in the initial screenshot. Do not solve this with coordinate scrolling. Use the generic Stage C semantic shell scroll surface.

## Current contract

- `get_buttons(scope="shell.scroll")` should expose semantic scroll controls for the active shell content root:
  - `shell.scroll.up`
  - `shell.scroll.down`
  - `shell.scroll.top`
  - `shell.scroll.bottom`
- Controls are `kind: scroll`, `scope: shell.scroll`.
- The control/state envelope should include safe position telemetry such as `y`, `max_y`, `viewport_height`, `can_scroll_up`, and `can_scroll_down`.
- `click_button(id="shell.scroll.down")` should animate the active content scrollable, then follow-up state/buttons should show a changed `y` or changed `can_scroll_*` values.

## Mission Control proof recipe

1. Navigate via MCP/open-tab to `missionControl` and verify the semantic state is mounted/authenticated/no blocking modal.
2. Capture the first visible state if it shows the claimed surface.
3. If the unified archive/history panel is below the fold, call `get_buttons(scope="shell.scroll")` and record the current `state.y/max_y/can_scroll_down`.
4. Use `click_button(id="shell.scroll.down")` or `shell.scroll.bottom`; do not coordinate-scroll.
5. Verify scroll state changed before claiming the interaction worked.
6. Capture visual proof after the scroll. If Win32/PrintWindow pixels are blank but semantic gates pass, use the Marionette internal Flutter screenshot fallback and label it as such.
7. For archive UX claims, the proof should show the unified lifecycle panel: `Ready for Archive`, `Mission Playback`, and `Archive Controls` in one coherent area, plus `View Archive` visibly revealing `Archived Missions`/history without hiding the rest of the lifecycle context.

## Regression signal

If `get_buttons(scope="shell.scroll")` is missing on a content-rich shell page, classify that as a Stage C semantic-control regression/gap. Do not fall back to coordinate scrolling as if it were acceptable evidence.
