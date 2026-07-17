# Mission Control video recording + live Harness goal smoke

Use this when Tony asks to record the Launcher while running a Mission Control / Agent Runtime Harness test goal.

## Pattern

1. Bring the Launcher window foreground/maximized first.
2. Record the visible desktop/window area with a bounded FFmpeg duration so the MP4 finalizes cleanly:
   ```bash
   mkdir -p '/c/Users/beast/AppData/Local/EterniaLauncher/stagec-smoke-local/videos'
   ffmpeg -y \
     -f gdigrab -framerate 15 \
     -offset_x 0 -offset_y 0 -video_size 2560x1440 \
     -i desktop \
     -t 75 \
     -c:v libx264 -preset ultrafast -pix_fmt yuv420p \
     '/c/Users/beast/AppData/Local/EterniaLauncher/stagec-smoke-local/videos/mission_control_test_goal_<timestamp>.mp4'
   ```
3. While recording, create a tiny Harness smoke goal and run it with `harness run-until-settled` using a strict action/time budget.
4. After FFmpeg exits, verify the MP4 container before sending:
   ```bash
   ffprobe -v error -show_entries format=duration,size -of default=noprint_wrappers=1:nokey=0 '<video>.mp4'
   ```
5. Send the MP4 via `MEDIA:<absolute path>` and summarize only the final task state / blocker.

## Important pitfall

Do **not** force-kill FFmpeg and then deliver the MP4. A killed FFmpeg process can leave `moov atom not found` / invalid MP4 output. If interactive stdin is unavailable, prefer `-t <seconds>` up front so FFmpeg exits normally and writes the final container metadata.

## Harness smoke goal notes

- Keep the goal explicitly read-only/no-code when the purpose is visual proof of the cockpit.
- Include a concrete affected repo/workdir in the goal description when Dev should run repo-grounded proof.
- A blocked result can still be useful video evidence if it exposes a truthful Harness intervention and leaves no active runs.
- Verify `harness status --json` or equivalent compact status after the recording; avoid repeated huge observe dumps unless debugging a specific stuck state.
