# Mission Control video proof and snapshot parity

Session lesson: a screen recording of a Harness goal run is **not** valid Mission Control UI proof unless the Launcher visibly shows the mission cards/counts changing during the recording.

## Acceptance pattern

Before claiming a Mission Control recording proves the UI:

1. Verify Harness CLI truth immediately before/during recording:
   - `hermes harness snapshot --json` or `hermes harness status --json`
   - record safe counts: `open_tasks`, `blocked_tasks`, `running_runs`, `open_incidents`
   - record task IDs/states only if safe
2. Verify Launcher is on `missionControl` through marionette navigation state.
3. Capture a still screenshot or inspect live pixels before recording and confirm the UI shows the same mission counts/cards expected from the CLI snapshot.
4. Start recording only after the UI parity check passes.
5. Run/create the test goal.
6. After recording, verify the final UI screenshot again. The video is accepted only if the active/blocked mission card or count is visible at some point.

## Failure classification

If Harness CLI shows open/blocked tasks but Launcher says `0 active missions` / empty state:

- Do **not** call the video a UI proof.
- Classify as a high-severity Mission Control live bridge/read-model parity bug.
- Report both sides explicitly:
  - Harness truth: task count/states from CLI.
  - Launcher truth: visible empty state / zero counts.
- Treat the recording as Harness execution evidence only, not Mission Control visibility evidence.

## Recording mechanics

For Windows foreground recordings, prefer a bounded-duration FFmpeg run so MP4 finalizes cleanly:

```bash
ffmpeg -y -f gdigrab -framerate 15 -offset_x 0 -offset_y 0 -video_size 2560x1440 -i desktop -t 75 -c:v libx264 -preset ultrafast -pix_fmt yuv420p output.mp4
```

Avoid killing FFmpeg when possible; force-killing can leave an invalid MP4 (`moov atom not found`). If early stop is required, use a recording method with controllable stdin/termination or choose a short bounded duration and re-record.

## User-facing rule

When Tony says the video does not show active missions, accept the correction immediately, verify parity, and retract the proof claim. Do not defend the recording based on CLI success alone.