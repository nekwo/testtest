Hermes Windows ops: Tony wants parallel Windows-native testing before leaving WSL. Git Bash uses POSIX quoting; wrap PowerShell `-Command` in single quotes so `$_` isn’t expanded by bash. Use `venv/Scripts`. `aliceimagecron` isolates image cron `a5803c276895`.
§
Eternia Backend testing: `scripts/test.sh` Postgres Docker full gate is required before backend push/deploy-readiness claims; targeted tests are not enough. `--infra-only --keycloak` starts local stack. SQLite is only Tony-requested escape hatch; if WSL full gate hangs, use GitHub Actions as authoritative but don’t claim local PASS.
§
Agent harness: Mission Control uses autonomous goal→Neko Lead→Dev→QA→proof→done/intervention, not Kanban/manual babysitting. Dev/QA stay independent, Dev sessions must be repo-grounded with project context, and proof gates/visibility come before UI polish.
§
Tony’s agent ops: Harness/Mission Control first; observability/auditability before automation fixes. Dev needs commits; QA needs screenshots/video/tests; recover in place.
§
Hermes worker skill sync: `kanban-orchestrator` sync script covers pm, QA, GPT/Claude, reviewer, and Spark profiles. After patching Alice workflow skills, run it and grep target profiles because copies don’t auto-update.
§
Eternia QA/env: Launcher default `.env` is production (prod Bob 8888/8889). Staging/local/QA use `.env.staging`/temp env/`STAGEC_*`, preserve YouTube/Twitch/Steam/IGDB keys, keep Stage34 loopback vs hosted Stage C separate. Gate: `getRuntimeState.ok=true`.
§
Stage C QA: use MCP semantic IDs; `screenshot_window` captures pixels. Mission Control smoke env can be env-routing/cooked: Launcher may show 0 missions while Alice CLI sees Harness tasks; verify Launcher runtime root/env before blaming UI polish. Kubeconfig may reset biweekly.
§
ArcadiaLabs_Brain is the shared operative layer for company/partner agents; TonyBrain stays personal. Shared brain links use relative paths; personal parent paths stay gitignored.
§
Education/OCW: Explore “Add to Eternia” must be one-click use-as-is via server pipeline into Library, not URL copy/Studio import. Prefer reusing existing carousel logic; duplicate only last resort.