# Kubernetes YAML Manifest Examples

This directory contains learning examples for the cluster lab. These are **templates to review and adapt**, not a claim that every object is currently applied in the live cluster.

## Apply safely

Run commands from the repository root and inspect the manifest before applying it.

```bash
kubectl apply --dry-run=client -f manifests/
kubectl diff -f manifests/
```

The directory contains examples with dependencies and environment-specific assumptions. Apply individual files in the order below rather than applying the whole directory blindly:

1. `namespace.yaml`
2. `nginx-deployment.yaml`
3. `nginx-service.yaml`
4. `app-configmap.yaml` (independent example)
5. `restricted-pod.yaml`
6. `storage-pv-pvc.yaml` (prepare the hostPath on the correct worker first)
7. `networkpolicy.yaml` (test in an isolated namespace; requires a NetworkPolicy-capable CNI)
8. `rbac-readonly.yaml`
9. `ingress.yaml` (requires a working Ingress Controller and the referenced Service)

The Secret file is deliberately a **template** containing placeholder values. Do not commit real credentials. Prefer creating Secrets from a protected local file or secret manager.

## Concepts in brief

- **Manifest**: declarative YAML describing the desired state of a Kubernetes API object.
- **apiVersion / kind**: API group/version and object type.
- **metadata**: object identity, including name, namespace and labels.
- **spec**: desired configuration. Controllers continuously reconcile actual state toward it.
- **Labels and selectors**: labels identify related objects; selectors connect a Service to matching Pods.
- **Deployment**: manages ReplicaSets and keeps the requested number of interchangeable Pods running.
- **Service**: stable virtual endpoint for a changing set of Pod IP addresses.
- **ConfigMap / Secret**: separate configuration from container images; Secret data still needs access controls and encryption-at-rest configuration.
- **PV / PVC**: storage resource and workload storage request. The hostPath example is node-local, not production-grade shared storage.
- **NetworkPolicy**: policy for Pod traffic. It has no effect unless the installed network plugin enforces it.
- **RBAC**: permissions are granted to identities through Role/ClusterRole and RoleBinding/ClusterRoleBinding.
- **Ingress**: HTTP routing rules interpreted by an installed Ingress Controller.

## Validation and cleanup

```bash
kubectl get all -n k8s-demo
kubectl describe deployment nginx -n k8s-demo
kubectl get endpointslice -n k8s-demo
kubectl delete -f manifests/nginx-service.yaml
kubectl delete -f manifests/nginx-deployment.yaml
kubectl delete namespace k8s-demo
```

Only delete objects you created for this exercise. The storage, NetworkPolicy, RBAC and Ingress examples need their own review before cleanup or use.
