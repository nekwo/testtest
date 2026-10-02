# Stage C blank visual after semantic navigation

> **Rewritten 2026-10-02** for the in-app capture lane (owner ruling
> OR-2026-10-02-qa-never-covers-the-screen). The desktop diagnostics this note used to
> prescribe — PrintWindow, a foreground `CopyFromScreen` — are retired: they put a QA
> Launcher over the owner's work. The classification below is the durable content.

## Trigger

Stage C MCP reports a healthy app/session/navigation state, but the capture is refused or
the PNG is blank or nearly uniform.

## How a blank frame arrives now

Every capture is the app's own rendered frame (`captureFrame`), judged by a content gate.
A blank frame does not come back as a PNG to inspect; it comes back as a named refusal:

- `capture_blank_frame` — the frame is uniform: nothing painted.
- `capture_low_information_frame` — a degenerate palette on a mostly-blank frame. Can be
  honest: the `stagec-smoke` account follows no communities, so Posts **For You** is a
  genuinely near-empty page. Navigate to a surface that has content (Posts **Explore**).
- `capture_frame_failed` — the app could not render its own frame (a `reason` rides
  along), or the bus was unreachable.
- `capture_in_app_unsupported` — the binary predates the in-app lane. Rebuild; there is no
  desktop fallback.

## Correct classification

Do **not** accept semantic MCP state as visual proof when the capture is refused. Treat
semantic-vs-visual disagreement as a high-severity visual QA blocker until a non-blank
in-app frame proves the rendered UI.

## Diagnostic sequence

1. If launch fails with `launch_wrong_debug_target_missing_marionette`, the binary is not a
   QA build. Let `launch_or_attach` rebuild its isolated QA copy (it does so when the copy
   is stale), or rebuild the marionette target per the Stage C operator runbook.
2. Relaunch/navigate with MCP and verify state: `navigation_state.selected_tab=<target>`,
   `blocking_modal_present=false`, shell mounted, expected semantic nav controls visible,
   and `auth_state.status=authenticated` when the surface needs a signed-in session
   (`dev_login` first).
3. Capture with `screenshot_window`.
4. On a refusal, switch to a known-good tab such as Home once and capture again. If that is
   refused too, classify it as a rendered-visual blocker rather than a surface-specific
   design issue.
5. Never raise, focus, restore or maximize the window to "help" a capture, and never take
   a desktop capture as a diagnostic.

## Reporting language

- "MCP semantic state is healthy, but the in-app capture was refused
  (`capture_blank_frame`). This is a FAIL/intervention, not an accepted screenshot."

Avoid saying the screenshot "looks good" or that visual QA passed when only semantic state
passed.
