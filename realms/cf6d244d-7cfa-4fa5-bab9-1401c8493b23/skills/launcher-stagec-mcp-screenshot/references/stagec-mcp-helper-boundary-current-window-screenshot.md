# Stage C MCP helper boundary + current-window screenshot sequence

Use when Tony asks for a Launcher screenshot and the current chat may not expose fresh first-class MCP tools.

## Boundary rule

The bounded operator helper `docs/stages/qa-reboot/scripts/Invoke-LauncherQaMcpTool.ps1` is still an MCP path: it starts/calls the Stage C MCP server with one bounded request, current config/env, timeout handling, and safe envelopes. Do not describe it as “not MCP” or as a plain PowerShell replacement for MCP.

Correct distinction:

- **Native in-chat MCP namespace**: MCP tools already loaded into the current Hermes conversation. These may retain stale env/config until the MCP client/session is restarted by the host.
- **Bounded operator MCP helper**: fresh MCP server invocation launched via PowerShell. Appropriate when the in-chat namespace is stale/closed or the chat lacks the project MCP tools.

## Correct screenshot flow

Even through the helper, use MCP end-to-end:

1. `mcp_launcher_qa_open_app_tab` — launch/attach, login if needed, navigate to requested tab.
2. Verify `ok=true`, `auth_state.status=authenticated`, requested `navigation_state.selected_tab`, `app.window_visible=true`, `app.qa_control_ready=true`.
3. `mcp_launcher_qa_screenshot_window` — capture current window state.
4. Send `MEDIA:<image_path>`.

Validated Library sequence:

```bash
powershell.exe -NoProfile -ExecutionPolicy Bypass \
  -File docs/stages/qa-reboot/scripts/Invoke-LauncherQaMcpTool.ps1 \
  -Tool mcp_launcher_qa_open_app_tab \
  -ArgsJson '{"tab":"library","browser_login":true,"credential_profile":"stagec-smoke","screenshot":false,"reap_stale":true}' \
  -CallTimeoutSeconds 240

powershell.exe -NoProfile -ExecutionPolicy Bypass \
  -File docs/stages/qa-reboot/scripts/Invoke-LauncherQaMcpTool.ps1 \
  -Tool mcp_launcher_qa_screenshot_window \
  -ArgsJson '{"window_title_prefix":"Eternia Launcher","label":"alice_library_current","out_dir":"X:/tmp/stagec/screenshots"}' \
  -CallTimeoutSeconds 120
```

Do not spend the screenshot request filesystem-searching helpers or hand-driving direct-control scripts first. Use the MCP tool sequence.
