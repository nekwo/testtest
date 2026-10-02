# In-app capture and evidence policy

Replaces `fullscreen-screenshot-and-video-evidence-policy.md` (retired 2026-10-02).

## Trigger

Any Launcher / Stage C / Mission Control visual proof — a screenshot, a sequence of
frames, or a request for video.

## The rule

Owner ruling OR-2026-10-02-qa-never-covers-the-screen: a QA Launcher never blocks the
owner's screen. It does not open on top of other windows, take focus, restore or
foreground itself for a capture, or open maximized over the owner's work. Screenshots are
rendered inside the app from its own frame. A capture that cannot be taken internally
fails with a named reason; it never falls back to foregrounding the window.

So, never:
- bring the Launcher window to the foreground, focus, restore or maximize it;
- take a desktop capture (PrintWindow, `CopyFromScreen`, `ImageGrab`, Snipping Tool, an
  FFmpeg `gdigrab` of the window title);
- reach for the VM-service `ext.flutter.marionette.takeScreenshots` path.

## How capture works

`mcp_launcher_qa_screenshot_window` and `mcp_launcher_qa_capture_screenshot` (and
`open_app_tab(screenshot: true)`) call the app's `captureFrame` verb: the root layer tree
is rasterized offscreen at the window's device pixel ratio and written to a PNG. The window
may sit behind other windows, the display may be asleep and the session locked. The
envelope carries `image_path`, physical `width`/`height`, `device_pixel_ratio`,
`frame_source`, method `in_app_frame`, and the content gate's verdict.

Window size is the app's own business: `launch_or_attach` opens at the default
1280x800 proof window (or the owner's size with `parity_window: true`) without covering
anything. `resize_window` is a debug control, not a capture prerequisite.

## Evidence defaults

- PNG for model/vision analysis and for human proof.
- Motion/timing defects: an in-app frame sequence with a state readback per frame, and a
  plain statement that no video was taken — there is no in-app recorder yet.
- If an older MP4 exists and the question is static, ignore it and capture a fresh frame.

## Operational sequence

1. Launch/attach and, if the surface needs a signed-in session, `dev_login` first.
2. Navigate and drive semantic controls; read the state back.
3. Capture with `screenshot_window`.
4. On a named refusal, classify it (see `stagec-blank-visual-after-semantic-nav.md`) —
   never retry by raising the window.
5. Deliver `MEDIA:<absolute png path>` first; keep analysis concise unless asked.
