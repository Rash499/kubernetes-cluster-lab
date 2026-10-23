# 07 — ConfigMaps and Secrets

## ConfigMaps
ConfigMaps hold non-sensitive configuration separately from the container image. The lab created app-config with APP_NAME, APP_ENV and APP_VERSION and injected values into a test Pod.

```bash
kubectl create configmap app-config --from-literal=APP_NAME=kubernetes-demo --from-literal=APP_ENV=development --from-literal=APP_VERSION=1.0
kubectl get configmap app-config
kubectl describe configmap app-config
```

## Secrets
Secrets hold sensitive configuration such as credentials. The lab tested db-secret and secretKeyRef. Example with placeholders only:
```bash
kubectl create secret generic db-secret --from-literal=DB_USERNAME='<username>' --from-literal=DB_PASSWORD='<password>'
kubectl get secret db-secret
kubectl describe secret db-secret
```

## Security
Base64 is encoding, not encryption. Restrict Secret access with least-privilege RBAC, configure encryption at rest where appropriate, and never commit Secret values, kubeconfigs, tokens or private keys. Avoid printing decoded data in terminal logs.