# 07 — ConfigMaps and Secrets

## Why separate configuration?

Keeping configuration outside the container image lets the same image run in development, testing and production with different settings. A ConfigMap is for **non-sensitive** values; a Secret is intended for sensitive values such as passwords, API tokens and certificates.

## ConfigMap

The lab created `app-config` with `APP_NAME`, `APP_ENV` and `APP_VERSION`. You can create it imperatively or apply [app-configmap.yaml](../manifests/app-configmap.yaml).

```bash
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/app-configmap.yaml
kubectl get configmap app-config -n k8s-demo
kubectl describe configmap app-config -n k8s-demo
```

A workload can consume ConfigMap values as environment variables or mount them as files. Environment variables are typically read when a container starts; changing the ConfigMap does not automatically change the environment of an already-running process. Mounted ConfigMap files update eventually, but the application may still need to reload them.

## Secret

The lab also tested `db-secret` and `secretKeyRef`. The repository includes [secret-template.yaml](../manifests/secret-template.yaml) only as a template. Its values are placeholders; never replace them with real credentials in a public commit.

For a local lab, create a Secret interactively so credentials do not become part of the shell command history:
```bash
kubectl create secret generic db-secret -n k8s-demo \
  --from-literal=DB_USERNAME='<username>' \
  --from-literal=DB_PASSWORD='<password>'
kubectl get secret db-secret -n k8s-demo
kubectl describe secret db-secret -n k8s-demo
```
Use placeholders in documentation. Be aware that command-line values can still be exposed through shell history or process/audit logs in some environments; a protected file or secret manager is preferable for sensitive systems.

## Important security details

- Base64 is an encoding format, **not encryption**.
- Kubernetes Secrets may be stored unencrypted in etcd unless encryption at rest is configured.
- Limit who can read Secrets with least-privilege RBAC; namespace boundaries alone do not make a Secret safe from all privileged users.
- Avoid printing Secret values or decoded data to logs.
- Do not commit kubeconfig files, tokens, private keys or real Secret manifests.
- For production, consider an external secret manager and a controlled secret-delivery workflow.

## Validation

```bash
kubectl get configmap,secret -n k8s-demo
kubectl get pod -n k8s-demo -o yaml
```
Inspect only the references and environment variable names when reviewing Pod YAML; avoid dumping sensitive values to shared logs.
