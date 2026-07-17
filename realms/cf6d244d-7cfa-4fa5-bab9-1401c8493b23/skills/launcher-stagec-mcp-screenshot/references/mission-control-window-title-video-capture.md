# Mission Control window-title video capture fallback

## Trigger

Use this when Tony asks for Mission Control video proof and desktop-level `gdigrab -i desktop` might capture the wrong foreground app (browser, X/Twitter, YouTube, etc.) despite the Launcher being open.

## Durable lesson

For user-facing Mission Control videos, prefer capturing the Launcher window by exact Win32 title after login/navigation/maximize:

```bash
ffmpeg -y \
  -f gdigrab -framerate 15 \
  -i title='Eternia Launcher (stagec-smoke)' \
  -t 50 \
  -c:v libx264 -preset ultrafast -pix_fmt yuv420p \
  '/c/Users/beast/AppData/Local/EterniaLauncher/stagec-smoke-local/videos/mission_control_window_test_goal_<timestamp>.mp4'
```

This avoids recording whatever browser window currently owns the desktop foreground. If the title form fails, enumerate real window titles first with PowerShell and use the exact title string:

```bash
powershell.exe -NoProfile -Command 'Get-Process | Where-Object { $_.MainWindowTitle } | Select-Object ProcessName,Id,MainWindowTitle | Sort-Object ProcessName | ConvertTo-Json -Depth 2'
```

Remember Hermes terminal is Git Bash/MSYS: wrap PowerShell `-Command` bodies in single quotes when they contain `$_` so Bash does not expand it.

## Preflight / verification sequence

1. Launch/authenticate via Stage C MCP and navigate to Mission Control; if the wrapper lacks `missionControl`, use the existing QA REST `/qa/setTab` fallback.
2. Maximize/fullscreen the Launcher window.
3. Take a short 3–5 second window-title sample capture and extract one frame.
4. Inspect only that single frame (not the full video) to verify it is actually Mission Control.
5. Start the bounded production capture with `-t`; never force-kill FFmpeg for deliverable MP4s.
6. During capture, run the Harness goal/tick.
7. Verify the final MP4 with `ffprobe` and sample one frame for sanity.
8. If the frame shows `0 active missions / 0 active runs` while CLI reports open tasks/runs, report it as a bridge/read-model parity issue, not a clean goal-in-action pass.

## Cleanup

Delete failed desktop-capture attempts that recorded unrelated apps before delivering proof. Keep the final window-title MP4 and one sampled frame when useful for evidence.
