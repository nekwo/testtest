# Stage C Smoke Credential Preflight: Windows kubeconfig and Rancher CA mismatch

Use this when `mcp_launcher_qa_open_app_tab(... browser_login=true, credential_profile=stagec-smoke ...)` or `Invoke-StageCBrowserLogin.ps1` fails with `auth_secret_unavailable`, but the Stage C smoke credential was previously provisioned for agents.

## Safe diagnosis

From the Windows/Hermes Git Bash environment, first prove this is credential-source access, not screenshot tooling:

```bash
kubectl config current-context
kubectl auth can-i get secret/stagec-smoke-credentials -n eternia-staging
kubectl -n eternia-staging get secret stagec-smoke-credentials -o go-template='{{.metadata.name}}{{" keys="}}{{range $k, $v := .data}}{{$k}} {{end}}{{"\n"}}'
```

Healthy result:

```text
local
yes
stagec-smoke-credentials keys=password username
```

Do **not** print secret `.data` values or decoded username/password.

## Common Rancher kubeconfig reset symptom

Tony's active Windows kubeconfig is:

```text
C:\Users\beast\.kube\config
```

When Rancher refresh/reset rewrites this file, the new kubeconfig may contain `certificate-authority-data` for the Rancher proxy endpoint:

```text
https://rancher.eternia.co/k8s/clusters/local
```

If kubectl then fails with:

```text
tls: failed to verify certificate: x509: certificate signed by unknown authority
```

but `curl https://rancher.eternia.co` verifies normally and `kubectl --insecure-skip-tls-verify=true ...` can read the Stage C secret, classify it as a Rancher kubeconfig CA mismatch, not missing Stage C credentials.

## Pragmatic fix

Restore the previously working cluster setting:

```bash
kubectl config set-cluster local --insecure-skip-tls-verify=true
```

Then rerun the safe diagnosis commands above. This keeps HTTPS transport but skips local certificate-chain verification for the Rancher proxy endpoint. For an enterprise-grade cleanup later, fix Rancher's advertised CA / kubeconfig generation so certificate verification works without this flag.

## Agent workflow rule

If screenshot QA is blocked by `auth_secret_unavailable`, say precisely:

- screenshot tooling is available;
- `stagec-smoke` credential path exists by contract;
- the active runner cannot currently access the secret until Windows kubeconfig/Rancher auth/TLS preflight passes.

Do not claim live Stage C screenshot QA passed until browser login and `getAuthState` prove authenticated state, then `screenshot_window` produces non-blank PNG evidence.
