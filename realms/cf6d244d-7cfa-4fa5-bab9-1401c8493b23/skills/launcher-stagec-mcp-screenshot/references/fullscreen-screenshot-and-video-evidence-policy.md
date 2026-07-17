# Fullscreen screenshots and video evidence policy

## Trigger

Use this for Launcher / Stage C / Mission Control visual proof where Tony asks for screenshots or video.

## Durable lesson

Tony wants the Launcher window made fullscreen/maximized before screenshots and visual evidence captures. Do this before live PNG proof and before starting bounded FFmpeg video capture.

## Evidence defaults

- Prefer PNG screenshots or a deliberately sampled key frame for model/vision analysis.
- Keep MP4/video artifacts as human proof by default.
- Only run full video analysis when the actual defect depends on motion, timing, transitions, loading, flicker, or live state changes.
- If a video already exists and the question is static/visual, sample one frame instead of analyzing the entire video.

## Operational sequence

1. Reopen/attach Launcher and navigate to the intended surface.
2. Bring the Launcher window foreground and maximize/fullscreen it before any screenshot or recording.
3. Capture PNG proof or start the bounded video capture.
4. If recording video, verify the MP4 with `ffprobe` before delivery.
5. For user-facing reporting, send the artifact first and keep analysis concise unless Tony asks for deeper inspection.
