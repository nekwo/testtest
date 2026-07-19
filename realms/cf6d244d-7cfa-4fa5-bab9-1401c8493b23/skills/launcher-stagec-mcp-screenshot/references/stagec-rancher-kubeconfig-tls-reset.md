# Stage C Rancher kubeconfig TLS/reset pitfall

Use this when Stage C MCP/browser-login screenshot QA reports `auth_secret_unavailable` even though the `stagec-smoke` agent credentials were previously provisioned.

## Symptom

- `mcp_launcher_qa_open_app_tab(... browser_login=true, credential_profile=stagec-smoke ...)` fails in browser login with `auth_secret_unavailable`.
- The helper reports Kubernetes read failure for `eternia-staging/stagec-smoke-credentials`, Credential Manager fallback absent, and interactive fallback unavailable in noninteractive PowerShell.
- `kubectl` exists on Windows and resolves from Git Bash, but access to Rancher fails.

## Correct diagnosis sequence

1. First distinguish missing `kubectl` from stale/reset kubeconfig:

   ```bash
   command -v kubectl
   kubectl config current-context
   kubectl config view --minify --raw
   ```

   Redact `token`, `certificate-authority-data`, `client-key-data`, `password`, etc. before sharing output.

2. Tony's active Windows kubeconfig path is normally:

   ```text
   C:\Users\beast\.kube\config
   ```

   From Git Bash/Hermes:

   ```text
   /c/Users/beast/.kube/config
   ```

   Tony reports this file may reset about every two weeks.

3. Check redaction-safe access without printing secret values:

   ```bash
   kubectl auth can-i get secret/stagec-smoke-credentials -n eternia-staging
   kubectl -n eternia-staging get secret stagec-smoke-credentials -o name
   kubectl -n eternia-staging get secret stagec-smoke-credentials \
     -o go-template='{{.metadata.name}}{{" keys="}}{{range $k, $v := .data}}{{$k}} {{end}}{{"\n"}}'
   ```

   Healthy output includes `yes`, `secret/stagec-smoke-credentials`, and keys `password username`.

## TLS reset failure

A freshly downloaded Rancher kubeconfig may contain `certificate-authority-data` for the cluster while the API server endpoint is the Rancher proxy URL:

```text
https://rancher.eternia.co/k8s/clusters/local
```

If `kubectl` fails with:

```text
tls: failed to verify certificate: x509: certificate signed by unknown authority
```

but succeeds with `--insecure-skip-tls-verify=true`, the token is probably valid and the embedded CA is mismatched for the current Rancher/Cloudflare/public TLS path.

Pragmatic fix used in-session:

```bash
kubectl config set-cluster local --insecure-skip-tls-verify=true
```

Then re-run the safe access checks above.

## AAA note

This is acceptable as a pragmatic Stage C QA unblock, matching the previously working config, but the enterprise-grade cleanup is to make Rancher generate kubeconfig CA data that validates the served Rancher proxy certificate chain or otherwise fix Rancher's advertised CA. Do not claim the secret is missing until kube auth/TLS has been checked.
