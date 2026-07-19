# Mission Control Launcher QA Smoke env parity

## Trigger

Use this when Mission Control/Agent Runtime Harness UI proof is captured through the Stage C / Launcher QA Smoke profile and the Launcher shows an empty Active Missions state while the Harness CLI reports open, blocked, or running work.

## Durable lesson

Do not classify this as a harmless empty state or UI polish pass. It is a live bridge/read-model parity failure until proven otherwise.

Mission Control proof has three distinct truth states:

1. **Real empty** — Harness summary reports `open_tasks == 0` and Launcher maps `0` active goals.
2. **Bridge unavailable/command failed** — Launcher could not read/decode the Harness snapshot; UI should surface a bridge diagnostic, not empty state.
3. **Parity mismatch** — Harness summary reports open/blocked/running work but Launcher maps `0` visible mission cards; UI should surface an intervention/diagnostic state.

## Env pinning pattern

For Launcher QA Smoke, the Launcher child process must read the same Harness runtime root as the CLI proof path. When using the Stage C direct-EXE helper, pin the child environment explicitly when the helper supports it:

- `HERMES_PROFILE=alice`
- `HERMES_AGENT_RUNTIME_ROOT=X:\Eternia\.hermes\agent-runtime`
- `HERMES_HOME=X:\Eternia\.hermes` when needed by the active profile/runtime

`Start-StageCDirectExe.ps1` supports optional parameters for this lane:

- `-HermesProfile`
- `-HarnessRuntimeRoot`
- `-HermesHome`

The safe launch envelope should include redaction-safe traceability fields such as `operator_label`, `hermes_profile`, `harness_runtime_root_configured`, and `hermes_home_configured`; never print secrets or raw tokens.

## Launch-chain audit pattern

When env pinning appears to be configured but the live Launcher still reads an empty or alternate Harness runtime, audit the **entire launch chain**, not only the PowerShell helper:

1. Confirm the helper script accepts and applies the env contract (`-HermesProfile`, `-HarnessRuntimeRoot`, `-HermesHome`) to the child Launcher process.
2. Inspect the MCP server launch layer that invokes the helper, especially `tool/stagec_qa_mcp_server/lib/launch_manager.dart` / `buildLaunchArgs()`. A common failure mode is that the helper supports the parameters but the MCP launch manager never forwards them.
3. Inspect MCP tool schemas/composers too: `mcp_launcher_qa_launch_or_attach` and `mcp_launcher_qa_open_app_tab` must accept `hermes_profile`, `harness_runtime_root`, and `hermes_home`, and `open_app_tab` must forward them into the inner launch/attach request.
4. Verify the resulting safe launch envelope, not just command intent. Required proof fields are `hermes_profile` non-null and `harness_runtime_root_configured=true`; if `hermes_profile:null` or `harness_runtime_root_configured:false`, the Launcher process is still unpinned.
5. Prove the Dart bridge separately when needed by running a targeted bridge/repository test against the live CLI snapshot. If the bridge maps non-empty CLI data correctly from the test process, focus the fix on process env/launch orchestration rather than Mission Control UI mapping.
6. Relaunch with stale-process cleanup after fixing env forwarding. An already-running unpinned Launcher can keep producing false empty UI even after the code is patched.

The implementation-ready fix for this class is usually to thread env-pinning fields through the MCP request/envelope into `LaunchManager.buildLaunchArgs()`, then into `Start-StageCDirectExe.ps1`, and finally into the child process environment. Add regression tests for the launch args and the open-tab/launch-or-attach composition path.

## Post-fix contract to preserve

A robust fix should include these durable guardrails:

- `LaunchRequest` carries optional `hermesProfile`, `harnessRuntimeRoot`, and `hermesHome` fields.
- `LaunchManager.buildLaunchArgs()` forwards non-empty values as `-HermesProfile`, `-HarnessRuntimeRoot`, and `-HermesHome`.
- `mcp_launcher_qa_launch_or_attach` and `mcp_launcher_qa_open_app_tab` schemas expose `hermes_profile`, `harness_runtime_root`, and `hermes_home`.
- `open_app_tab` forwards those values to its inner launch/attach path.
- Safe envelopes expose only redaction-safe status: `hermes_profile`, `harness_runtime_root_configured`, and `hermes_home_configured`.
- Pinned requests must not silently reuse an unpinned live disk session. Refuse with a bounded failure such as `app_env_pin_mismatch`, with a suggested next action to relaunch using `force_relaunch:true` / `reap_stale:true` and the requested pins.
- If the tab schema was stale, include `missionControl` in the Stage C MCP tab enums before relying on first-class `open_app_tab`/`set_tab` for that surface.
- After MCP source/tool-catalog changes, rebuild the canonical server executable: `tool/stagec_qa_mcp_server/build/stagec_qa_mcp_server.exe`.

Regression tests should cover at least:

1. `LaunchManager.buildLaunchArgs()` includes the three helper flags when supplied.
2. `launch_or_attach` accepts env pinning and records it on the launch request.
3. `open_app_tab` accepts env pinning and forwards it to launch/attach.
4. A pinned request refuses an unpinned live disk session rather than attaching silently.
5. Failure-class/suggested-next-action registries include the new bounded failure class.

## Proof checklist

Before accepting Mission Control screenshots/video as proof:

1. Capture CLI/Harness summary counts from the same intended runtime root.
2. Verify Launcher Mission Control renders matching active/blocked mission cards or, if it cannot, renders an explicit bridge diagnostic/intervention state.
3. Verify the launch/open envelope reports `hermes_profile` and `harness_runtime_root_configured=true` for pinned Launcher QA Smoke proof.
4. If using video, verify parity before and after recording; a successful Harness run with an empty Launcher UI is bug evidence, not UI proof.
5. If the wrapper schema rejects `missionControl`, a bounded QA REST `/qa/setTab` call is acceptable only when the live marionette port/nonce already exists; still use `mcp_launcher_qa_screenshot_window` for pixels.
6. Report exact gate scope: targeted tests/analyzer/build pass does not replace live CLI↔Launcher parity proof.

## Implementation pattern

In the Launcher read model/snapshot, carry a redaction-safe bridge diagnostics object with:

- bridge status enum, e.g. `ok`, `commandFailed`, `decodeFailed`, `parityMismatch`, `unknown`;
- CLI summary counts: open tasks, blocked tasks, active runs;
- mapped Launcher active goal count;
- safe operator/profile labels.

Render the bridge diagnostic state before the normal empty-state widget when diagnostics require attention.

## Related pitfalls

- Do not use screenshot/video as success evidence if it does not visibly show the exact claimed Mission Control state.
- Do not rename away legacy `stagec-smoke` mechanically; introduce `Launcher QA Smoke` as the operator-facing label/alias while preserving compatibility contracts until all scripts/tools are migrated.
- Process enumeration on Windows may throw when reading `StartTime`; helpers should tolerate access-denied metadata and use a safe fallback sort rather than crashing the launch path.
- Running `dart format lib test` in the MCP server package can reformat many unrelated files. Prefer formatting only touched files, then use `git diff --check` and `git diff --stat --ignore-space-at-eol` before committing.
