# Mission Control Smoke Pinning and Empty-Goals Parity

## Lesson

A Stage C smoke Mission Control run that shows no active goals is not enough evidence by itself. First prove whether the Launcher child process was launched with the intended Harness pins, then compare the exact pinned Harness store against the Launcher UI.

## Durable pattern

1. Inspect the launch/open envelope before interpreting Mission Control state.
   - Required for pinned Harness proof:
     - `app.hermes_profile` is the intended profile, usually `alice` for Tony/Alice proof.
     - `app.harness_runtime_root_configured == true`.
     - `app.hermes_home_configured == true` when the run depends on Alice profile config.
   - If any of these are missing/null, the smoke did **not** exercise the env-pinning path, even if the code supports it.

2. Run the CLI snapshot against the same intended runtime root/profile used by the Launcher child.
   - Compare `open_tasks`, terminal/non-terminal tasks, and mapped active goals.
   - A root with `open_tasks == 0` and only terminal tasks means an empty Active Missions UI may be correct for that root.

3. If Tony's normal launch shows goals while pinned smoke shows none, do not claim the UI is fixed or broken yet.
   - Treat it as a runtime-store/profile mismatch until the normal launch's `HERMES_PROFILE`, `HERMES_HOME`, and `HERMES_AGENT_RUNTIME_ROOT` are known.
   - Search likely Harness stores only to identify the alternate root; do not turn one observed root into a universal rule.

4. State the result precisely.
   - `Forwarding path works when explicit pins are supplied` is different from `the smoke recipe always uses the right pins`.
   - `Pinned root is genuinely empty` is different from `Mission Control failed to load goals`.

## Pitfall

Do not answer “we fixed that” from source-code support alone. A previous smoke launch envelope with `hermes_profile:null` or `harness_runtime_root_configured:false` proves the run was unpinned, so it cannot validate the fix. Re-run pinned, then verify CLI↔Launcher parity for that exact runtime root.

## Recommended acceptance

- Launch envelope proves pins.
- CLI snapshot from the same root is captured/parsed.
- Launcher UI active/archived/terminal state matches the CLI snapshot.
- If normal env differs, identify the normal env root/profile before patching UI mapping or declaring success.
