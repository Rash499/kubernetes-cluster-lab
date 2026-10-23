# 10 — Pod Security, RBAC and NetworkPolicy

These controls protect different layers. Pod security constrains what a workload can do; RBAC controls who can call the Kubernetes API; NetworkPolicy controls selected Pod network traffic when supported by the CNI.

## Pod Security Admission

The lab created namespace `pod-security-demo` and successfully ran `restricted-pod`. Pod Security Admission applies namespace-level standards: `privileged`, `baseline` and `restricted`. The Restricted profile is the most constrained of these built-in profiles and expects hardened Pod settings.

```bash
kubectl get namespace pod-security-demo --show-labels
kubectl get pod restricted-pod -n pod-security-demo
kubectl exec restricted-pod -n pod-security-demo -- id
```

The [restricted-pod.yaml](../manifests/restricted-pod.yaml) example demonstrates non-root execution, RuntimeDefault seccomp, dropped Linux capabilities, disabled privilege escalation, a read-only root filesystem and no automatic service-account token mount. It is a learning manifest; confirm image compatibility and validate in a dedicated namespace.

## RBAC: who can do what?

RBAC permissions are additive. A **Role** grants permissions within one namespace; a **ClusterRole** can define cluster-wide or reusable permissions. A **RoleBinding** grants a Role to users, groups or ServiceAccounts in one namespace. A **ClusterRoleBinding** grants a ClusterRole at cluster scope.

The [rbac-readonly.yaml](../manifests/rbac-readonly.yaml) example grants only `get`, `list` and `watch` on Pods in `k8s-demo`. The lab also tested a ClusterRole/ClusterRoleBinding for read-only node access and verified allowed/denied operations with `kubectl auth can-i`.

```bash
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/rbac-readonly.yaml
kubectl auth can-i list pods -n k8s-demo --as=system:serviceaccount:k8s-demo:pod-reader
kubectl auth can-i get secrets -n k8s-demo --as=system:serviceaccount:k8s-demo:pod-reader
kubectl get roles,rolebindings -n k8s-demo
kubectl get clusterroles,clusterrolebindings
```
The first check should be allowed and the Secret check denied, assuming the manifest was applied and the caller may impersonate the ServiceAccount. Never solve access problems by reflexively granting `cluster-admin`.

## NetworkPolicy: which Pods may communicate?

A NetworkPolicy selects Pods and declares allowed ingress and/or egress traffic. Once a selected Pod is isolated for a direction, traffic not allowed by applicable policies is denied for that direction. Policies are additive; allowed traffic is the union of matching rules. DNS egress can also be affected by egress policies.

The original lab created a dedicated test namespace, server and client Pods, a Service, checked baseline connectivity, applied default-deny ingress, confirmed traffic was blocked, then added a policy allowing a labelled client and confirmed an unlabelled client remained blocked.

See [networkpolicy.yaml](../manifests/networkpolicy.yaml) for a focused Nginx policy and [networkpolicy-isolation.yaml](../manifests/networkpolicy-isolation.yaml) for a default-deny plus allow-list pattern. Apply policy experiments only in a dedicated namespace after confirming CNI enforcement.

```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy -n k8s-demo
kubectl get pods -n k8s-demo --show-labels
kubectl get endpointslice -n k8s-demo
```

Later lab notes identify Calico v3.33.0 during NetworkPolicy exercises; the initial setup used Flannel. Verify the active CNI because a NetworkPolicy may be accepted by the API without being enforced by an incompatible network plugin.

## General security checklist

- Use least privilege and dedicated ServiceAccounts.
- Do not store credentials in ConfigMaps or commit real Secret values.
- Avoid privileged containers, host namespaces and unnecessary Linux capabilities.
- Keep images and packages updated and define resource requests/limits.
- Test both expected-allowed and expected-denied paths.
- Verify the actual applied Pod and namespace configuration rather than assuming YAML was accepted exactly as intended.
