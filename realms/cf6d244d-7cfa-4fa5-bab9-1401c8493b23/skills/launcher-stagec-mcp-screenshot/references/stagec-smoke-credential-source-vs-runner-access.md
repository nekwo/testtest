# Stage C smoke credentials: provisioned source vs active runner access

Use this when Stage C MCP screenshot/auth fails with `auth_secret_unavailable` and Tony reminds us that credentials were already made for agents.

## Durable distinction

Do **not** summarize this as "credentials do not exist" unless source probes prove the documented credential contract is absent.

The Stage C agent credential contract is:

- identity: `stagec-smoke`
- expected username: `qa-stagec-smoke`
- Kubernetes secret: `eternia-staging/stagec-smoke-credentials`
- Windows Credential Manager fallback target: `EterniaStageC/stagec-smoke`

A screenshot/login attempt can still fail when the active Hermes runner cannot access those sources:

- Kubernetes context present but not authenticated / expired.
- Windows Credential Manager fallback target absent for the current Windows user.
- Interactive fallback unavailable because helper runs non-interactively.

## Reporting language

Say:

> Screenshot QA tooling exists, and the agent credential path was designed/provisioned. The current runner cannot reach a credential source right now, so authenticated screenshot QA is blocked until kube auth or the local Credential Manager fallback is restored.

Avoid saying:

> We have no credentials.

## Verification sequence

1. Run the Stage C credential helper dry-run / safe probe when allowed by the active task context.
2. Treat `kubectl present` as only binary/context availability; a later real secret read can still fail if cluster auth is expired.
3. Check the Credential Manager target only by name/presence; never print password values.
4. If credential sources are unreachable, classify as an environment/credential-access blocker, not as missing screenshot tooling.

## Fix options to propose

- Refresh Kubernetes auth for the current runner/user so `eternia-staging/stagec-smoke-credentials` can be read.
- Populate Windows Credential Manager target `EterniaStageC/stagec-smoke` for this user.
- Reuse an already-authenticated cached Launcher session only when acceptable for the QA claim.

## Redaction rule

Never print, quote, store, or persist the password/token/callback-code material. Persist only source names, usernames, safe statuses, and failure classes.
