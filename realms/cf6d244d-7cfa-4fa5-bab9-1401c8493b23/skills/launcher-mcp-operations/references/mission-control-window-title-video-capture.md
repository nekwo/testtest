# Window-title video capture

> **Refresh note (2026-08-28):** the "run the Harness goal/tick during capture" step and
> the `0 active missions / 0 active runs` parity check are removed — the goal/task mission
> lane no longer exists. The window-title `gdigrab` capture and the bounded-duration MP4
> mechanics are the durable content, and this file now also carries the never-force-kill
> FFmpeg rule that used to live in the dropped video-recording note.

## Trigger

Use this when the operator asks for Launcher video proof and desktop-level
`gdigrab -i desktop` might capture the wrong foreground app (browser, X/Twitter, YouTube,
etc.) despite the Launcher being open.

## Durable lesson

For user-facing Launcher videos, prefer capturing the Launcher window by exact Win32 title
after login/navigation/maximize:

```bash
ffmpeg -y \
  -f gdigrab -framerate 15 \
  -i title='Eternia Launcher (stagec-smoke)' \
  -t 50 \
  -c:v libx264 -preset ultrafast -pix_fmt yuv420p \
  '/c/Users/beast/AppData/Local/EterniaLauncher/stagec-smoke-local/videos/launcher_window_<label>_<timestamp>.mp4'
```

This avoids recording whatever browser window currently owns the desktop foreground. If
the title form fails, enumerate real window titles first and use the exact title string:

```bash
powershell.exe -NoProfile -Command 'Get-Process | Where-Object { $_.MainWindowTitle } | Select-Object ProcessName,Id,MainWindowTitle | Sort-Object ProcessName | ConvertTo-Json -Depth 2'
```

Remember the Hermes terminal is Git Bash/MSYS: wrap PowerShell `-Command` bodies in single
quotes when they contain `$_` so Bash does not expand it.

Create the output directory first if it does not exist:

```bash
mkdir -p '/c/Users/beast/AppData/Local/EterniaLauncher/stagec-smoke-local/videos'
```

## Bounded duration, never force-kill

Always pass `-t <seconds>` up front so FFmpeg exits normally and writes the final
container metadata. Force-killing FFmpeg can leave an invalid MP4 that fails with
`moov atom not found`. If an early stop is required, choose a shorter bounded duration and
re-record rather than killing a running capture.

Verify the container before delivery:

```bash
ffprobe -v error -show_entries format=duration,size -of default=noprint_wrappers=1:nokey=0 '<video>.mp4'
```

## Preflight / verification sequence

1. Launch/authenticate via Stage C MCP and navigate to the target surface; if the wrapper
   lacks the tab enum value, use the existing QA REST `/qa/setTab` fallback.
2. Maximize/fullscreen the Launcher window.
3. Take a short 3-5 second window-title sample capture and extract one frame.
4. Inspect only that single frame (not the full video) to verify it is actually the
   Launcher on the intended surface.
5. Start the bounded production capture with `-t`; never force-kill FFmpeg for
   deliverable MP4s.
6. If a live action should be visible in the recording, drive it through the chat lane or
   semantic MCP controls while the capture runs.
7. Verify the final MP4 with `ffprobe` and sample one frame for sanity.
8. If the sampled frame does not show the state the recording claims, report it as a
   Launcher/read-model parity issue rather than presenting the MP4 as a clean pass.

## Cleanup

Delete failed desktop-capture attempts that recorded unrelated apps before delivering
proof. Keep the final window-title MP4 and one sampled frame when useful as evidence.
