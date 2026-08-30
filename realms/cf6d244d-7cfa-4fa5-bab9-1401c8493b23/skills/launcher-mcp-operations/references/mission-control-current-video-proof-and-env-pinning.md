# Current video proof and env pinning

> **Refresh note (2026-08-28):** mission-era acceptance ("CLI open-task counts match
> active/blocked mission cards") is removed — that lane no longer exists. The
> stale-MP4 recovery and key-frame rule are the durable content; compare against live
> surfaces instead: roster, chat transcript, board, office scene, agent console.

## Trigger

Use this when the operator asks for Launcher video/screenshot proof, or when a video
artifact does not show the same state as the current Launcher screenshot.

## Stale video artifacts are not proof

A previously recorded MP4 can be valid media but invalid proof if it predates the current
fix/state. If the operator says the screenshot shows the right UI but the video does not,
assume the MP4 is stale until proven otherwise. Do not defend or resend the old file.

Correct recovery:

1. Bring the Launcher window foreground and maximize/fullscreen it.
2. Record a fresh bounded MP4 of the exact Launcher window title, not the desktop:

   ```bash
   ffmpeg -y -f gdigrab -framerate 30 \
     -i title='Eternia Launcher (stagec-smoke)' \
     -t 20 -c:v libx264 -preset veryfast -pix_fmt yuv420p <out>.mp4
   ```

3. Sample a key frame from the new MP4:

   ```bash
   ffmpeg -y -ss 00:00:02 -i <out>.mp4 -frames:v 1 <keyframe>.png
   ```

4. Verify the key frame, not the MP4 by assumption.
5. Send the new MP4 only after `ffprobe` confirms duration/size **and** the key frame
   shows the claimed UI state.

## Env-pinned Launcher QA Smoke proof

Stage C proof is only trustworthy when the Launcher process is pinned to the same Harness
runtime the CLI is checking.

Required safe envelope signals for pinned proof:

- `app.hermes_profile: alice` or the intended profile
- `app.harness_runtime_root_configured: true`
- `app.hermes_home_configured: true`
- `navigation_state.selected_tab: <requested tab>`

The MCP path supports `hermes_profile`, `harness_runtime_root`, and `hermes_home`, and the
Stage C MCP launch manager forwards those to `Start-StageCDirectExe.ps1` as
`-HermesProfile`, `-HarnessRuntimeRoot`, and `-HermesHome`.

Pinned requests must refuse unpinned live disk sessions with a bounded
`app_env_pin_mismatch` instead of silently attaching to a stale Launcher that may show
an empty or foreign runtime's state.

## Acceptance pattern

A valid video proof has all of:

- a current fullscreen Launcher window capture;
- the visible Launcher surface matching the CLI/Harness state it claims (roster rows, chat
  transcript, board cards, office scene, agent console);
- a key-frame PNG showing the same state claimed for the MP4;
- an MP4 freshly recorded after the fix/state change, not reused from an earlier smoke.
