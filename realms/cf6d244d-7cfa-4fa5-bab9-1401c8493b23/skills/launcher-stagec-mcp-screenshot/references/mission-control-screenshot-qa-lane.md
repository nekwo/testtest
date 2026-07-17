# Mission Control screenshot QA lane

Use this note when validating Mission Control / Agent Runtime Harness UI changes in the Launcher.

## Durable lesson

Stage C screenshot QA is the standard visual proof lane for user-facing Launcher Mission Control work, but it should not block deterministic non-UI Harness/backend implementation runs. If it is skipped or unavailable, the final report must explicitly say:

> live Stage C screenshot QA not performed

Do not silently imply visual QA was done just because Flutter tests, Harness smoke tests, or semantic MCP state passed.

## Availability triage

When screenshot QA is unavailable, classify the reason precisely:

- `auth_secret_unavailable` / browser login cannot get credentials: credential-access blocker, usually active runner/kubeconfig access; follow the Stage C smoke credential preflight references. In recent Mission Control work, screenshot QA was unavailable because the active Windows runner had stale Rancher/kube auth/TLS access to `eternia-staging/stagec-smoke-credentials`, not because the screenshot MCP lane or Stage C credentials were absent. Verify with redaction-safe `kubectl auth can-i ...` and secret-name/key-name checks before declaring credentials missing.
- MCP state succeeds but screenshot is blank: visual/capture blocker; do not count as screenshot proof.
- Page-local semantic controls missing/stale: MCP schema/session freshness blocker; rebuild/reload or use bounded helper, but do not coordinate-click as proof.
- Debug Launcher binary is stale or locked: kill the running `eternia_launcher.exe`, rebuild the Windows debug/marionette target, relaunch through MCP, then capture a fresh PNG. Relaunching the app alone is not the same as completed screenshot QA unless a screenshot path is captured and reported.

## Mission Control proof checklist

For Mission Control UI changes, collect at least one real PNG proof when available:

1. Open the app through MCP with `credential_profile: stagec-smoke` and `browser_login: true`.
2. Navigate to Mission Control / relevant shell surface.
3. Drive semantic controls if exposed; otherwise record the semantic-control gap.
4. Capture with `mcp_launcher_qa_screenshot_window`.
5. Report the PNG path (`MEDIA:<path>` if sending to Tony) and the state assertions used.

## Reporting rule

Separate deterministic proof from live visual proof:

- Good: `Flutter tests passed; Harness smoke passed; live Stage C screenshot QA not performed because <reason>.`
- Bad: `QA passed` when no live screenshot was captured for a user-facing Launcher change.
