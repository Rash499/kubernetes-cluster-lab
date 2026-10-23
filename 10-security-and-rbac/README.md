# 10 — Pod Security, RBAC and NetworkPolicy

## Pod Security Admission
The lab created namespace pod-security-demo and successfully ran restricted-pod. Check current labels and Pod identity:
```bash
kubectl get namespace pod-security-demo --show-labels
kubectl get pod restricted-pod -n pod-security-demo
kubectl exec restricted-pod -n pod-security-demo -- id
```

Common Restricted-style security context:
```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
  seccompProfile:
    type: RuntimeDefault
```
Verify the actual Pod YAML before claiming every setting is applied.

## RBAC
RBAC grants permissions using Roles/ClusterRoles and bindings.
```bash
kubectl auth can-i get pods
kubectl get roles,rolebindings -A
kubectl get clusterroles,clusterrolebindings
```
The notes include a restricted kubeconfig and troubleshooting a localhost:8080 kubeconfig issue. Test with the intended identity and avoid broad cluster-admin grants.

## NetworkPolicy
Later notes cover NetworkPolicy and traffic-isolation tests and identify Calico as active at that time. Enforcement requires a CNI that supports NetworkPolicy.
```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy -n <namespace> <policy-name>
```

Test both allowed and denied paths, recording source, destination, namespace selectors and results. Use a dedicated test namespace and never commit credentials.