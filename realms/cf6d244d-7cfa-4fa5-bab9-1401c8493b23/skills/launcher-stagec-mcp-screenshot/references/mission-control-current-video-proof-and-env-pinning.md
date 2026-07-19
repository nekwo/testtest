# Mission Control current video proof + Launcher QA Smoke env pinning

## Trigger

Use this when Tony asks for Mission Control video/screenshot proof, or when a video artifact does not show the same state as the current Launcher screenshot.

## Lessons from the 2026-05-31 / 2026-06-01 Mission Control session

### Stale video artifacts are not proof

A previously recorded MP4 can be valid media but invalid proof if it predates the current fix/state. If Tony says the screenshot shows the right UI but the video does not, assume the MP4 is stale until proven otherwise.

Correct recovery:

1. Bring the Launcher window foreground and maximize/fullscreen it.
2. Record a fresh bounded MP4 of the exact Launcher window title, not the desktop:
   - `ffmpeg -y -f gdigrab -framerate 30 -i title='Eternia Launcher (stagec-smoke)' -t 20 -c:v libx264 -preset veryfast -pix_fmt yuv420p <out>.mp4`
3. Sample a key frame from the new MP4:
   - `ffmpeg -y -ss 00:00:02 -i <out>.mp4 -frames:v 1 <keyframe>.png`
4. Verify the key frame, not the MP4 by assumption.
5. Send the new MP4 only after `ffprobe` confirms duration/size and the key frame shows the claimed UI state.

### Env-pinned Launcher QA Smoke proof

Mission Control Stage C proof is only trustworthy when the Launcher process is pinned to the same Harness runtime that the CLI is checking.

Required safe envelope signals for pinned proof:

- `app.hermes_profile: alice` or the intended profile
- `app.harness_runtime_root_configured: true`
- `app.hermes_home_configured: true`
- `navigation_state.selected_tab: missionControl`

The post-fix MCP path supports:

- `hermes_profile`
- `harness_runtime_root`
- `hermes_home`

and the Stage C MCP launch manager forwards those to `Start-StageCDirectExe.ps1` as:

- `-HermesProfile`
- `-HarnessRuntimeRoot`
- `-HermesHome`

Pinned requests must refuse unpinned live disk sessions with a bounded `app_env_pin_mismatch` instead of silently attaching to a stale Launcher that may show `0 active missions`.

## Acceptance pattern

A valid Mission Control video proof has all of:

- current fullscreen Launcher window capture;
- CLI/Harness open-task state matches visible Mission Control cards/counts;
- key-frame PNG shows the same state claimed for the MP4;
- MP4 is freshly recorded after the fix/state change, not reused from an earlier smoke.
