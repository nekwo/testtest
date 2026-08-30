# Stage C Smoke Credential Preflight: Windows kubeconfig and Rancher CA mismatch

> **Merge note (2026-08-28):** this file absorbs the former
> `stagec-rancher-kubeconfig-tls-reset.md`. The two were near-duplicates; everything
> unique to the TLS-reset note (the `kubectl config view --minify --raw` redaction rule,
> the two-week kubeconfig reset cadence, the `command -v kubectl` missing-vs-stale split,
> and the AAA cleanup note) is preserved below.

Use this when `mcp_launcher_qa_open_app_tab(... browser_login=true, credential_profile=stagec-smoke ...)`
or `Invoke-StageCBrowserLogin.ps1` fails with `auth_secret_unavailable`, but the Stage C
smoke credential was previously provisioned for agents.

## Step 0 — missing tool vs stale config

First distinguish a missing `kubectl` from a stale/reset kubeconfig:

```bash
command -v kubectl
kubectl config current-context
kubectl config view --minify --raw
```

Redact `token`, `certificate-authority-data`, `client-key-data`, `password`, and similar
fields before sharing any of that output.

## Safe diagnosis

From the Windows/Hermes Git Bash environment, prove this is credential-source access, not
screenshot tooling:

```bash
kubectl config current-context
kubectl auth can-i get secret/stagec-smoke-credentials -n eternia-staging
kubectl -n eternia-staging get secret stagec-smoke-credentials -o name
kubectl -n eternia-staging get secret stagec-smoke-credentials \
  -o go-template='{{.metadata.name}}{{" keys="}}{{range $k, $v := .data}}{{$k}} {{end}}{{"\n"}}'
```

Healthy result:

```text
local
yes
secret/stagec-smoke-credentials
stagec-smoke-credentials keys=password username
```

Do **not** print secret `.data` values or decoded username/password.

## Common Rancher kubeconfig reset symptom

The active Windows kubeconfig is:

```text
C:\Users\beast\.kube\config
```

From Git Bash/Hermes:

```text
/c/Users/beast/.kube/config
```

The operator reports this file may reset about every two weeks. When Rancher
refresh/reset rewrites it, the new kubeconfig may carry `certificate-authority-data` for
the cluster while the API server endpoint is the Rancher proxy URL:

```text
https://rancher.eternia.co/k8s/clusters/local
```

If kubectl then fails with:

```text
tls: failed to verify certificate: x509: certificate signed by unknown authority
```

but `curl https://rancher.eternia.co` verifies normally and
`kubectl --insecure-skip-tls-verify=true ...` can read the Stage C secret, the token is
probably valid and the embedded CA is mismatched for the current
Rancher/Cloudflare/public TLS path. Classify it as a Rancher kubeconfig CA mismatch, not
missing Stage C credentials.

## Pragmatic fix

Restore the previously working cluster setting:

```bash
kubectl config set-cluster local --insecure-skip-tls-verify=true
```

Then rerun the safe diagnosis commands above. This keeps HTTPS transport but skips local
certificate-chain verification for the Rancher proxy endpoint.

## AAA note

The `--insecure-skip-tls-verify` setting is an acceptable pragmatic Stage C QA unblock
that matches the previously working config. The enterprise-grade cleanup is to make
Rancher generate kubeconfig CA data that validates the served Rancher proxy certificate
chain, or otherwise fix Rancher's advertised CA. Do not claim the secret is missing until
kube auth/TLS has been checked.

## Agent workflow rule

If screenshot QA is blocked by `auth_secret_unavailable`, say precisely:

- screenshot tooling is available;
- the `stagec-smoke` credential path exists by contract;
- the active runner cannot currently access the secret until Windows kubeconfig / Rancher
  auth / TLS preflight passes.

Do not claim live Stage C screenshot QA passed until browser login and `getAuthState`
prove authenticated state, then `screenshot_window` produces non-blank PNG evidence.
See `stagec-smoke-credential-source-vs-runner-access.md` for the reporting language that
separates a provisioned credential contract from runner access.
