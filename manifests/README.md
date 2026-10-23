# Kubernetes YAML Manifest Examples

This directory contains educational manifests distilled from the lab and additional small examples to make concepts reproducible. These are templates to review and adapt, not a claim that every object is currently applied in the live cluster.

## Important: do not apply the entire directory

Some manifests depend on optional controllers or cluster-specific resources. `kubectl apply -f manifests/` may fail or create resources you did not intend to run. Review and apply files individually.

## Manifest catalog

| File | Purpose | Prerequisites / cautions |
|---|---|---|
| `namespace.yaml` | Isolated demo namespace `k8s-demo` | Apply first |
| `nginx-deployment.yaml` | Three Nginx replicas, probes and resources | Standard Deployment |
| `nginx-service.yaml` | ClusterIP front end for Nginx | Requires matching Deployment labels |
| `app-configmap.yaml` | Non-sensitive environment configuration | Namespace must exist |
| `secret-template.yaml` | Secret format template with placeholders | Never commit real values |
| `restricted-pod.yaml` | Non-root container, seccomp, dropped capabilities, read-only root FS | Image/runtime compatibility may vary |
| `storage-pv-pvc.yaml` | Node-local PV and PVC | Requires correct worker path and matching node name |
| `networkpolicy.yaml` | Restricts Nginx ingress to labelled clients | Requires a CNI that enforces NetworkPolicy |
| `networkpolicy-isolation.yaml` | Default-deny ingress plus explicit allow-list | Use only in a dedicated test namespace |
| `rbac-readonly.yaml` | Namespace-scoped Pod read-only RBAC | Impersonation checks require permission |
| `ingress.yaml` | Host-based HTTP routing | Requires Ingress Controller and DNS/hosts entry |
| `hpa-demo.yaml`, `hpa.yaml` | Demo Deployment/Service and CPU HPA | Metrics Server and Metrics API must work |
| `resource-qos-demo.yaml` | CPU/memory requests and limits | Demonstration only |
| `high-cpu-alert.yaml` | Prometheus Operator alert rule | Requires PrometheusRule CRD and matching selectors |
| `probes-demo.yaml` | Startup, readiness and liveness probes | Demonstration Deployment and Service |
| `statefulset-demo.yaml` | StatefulSet with per-replica PVC templates | Requires a dynamic/default StorageClass |
| `daemonset-demo.yaml` | One simple agent Pod per eligible node | Scheduling depends on node taints |
| `job-cronjob-demo.yaml` | Finite Job and scheduled CronJob | CronJob repeats every 15 minutes until deleted/suspended |
| `toleration-demo.yaml` | Node selector plus matching toleration | Requires matching worker1 taint and hostname label |
| `metallb-l2-pool.yaml` | MetalLB IPAddressPool and L2Advertisement | Requires MetalLB CRDs; reserve the address range first |

A minimal Helm chart is in [`charts/web-demo`](../charts/web-demo/Chart.yaml), not in this directory.

## Recommended starting sequence

Run from the repository root:

```bash
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/nginx-deployment.yaml
kubectl apply -f manifests/nginx-service.yaml

kubectl rollout status deployment/nginx -n k8s-demo
kubectl get deployment,replicaset,pods,svc -n k8s-demo -o wide
kubectl get endpointslice -n k8s-demo -l kubernetes.io/service-name=nginx-service
```

For other examples, read their topic guide first:
- [Services and packet flow](../17-service-types-and-networking/README.md)
- [Observability](../14-observability/README.md)
- [Scaling and resources](../15-scaling-and-resources/README.md)
- [Helm](../16-helm/README.md)
- [Controllers and probes](../18-workload-controllers-and-probes/README.md)

## Core concepts

- **Manifest:** declarative desired state for an API object.
- **apiVersion / kind:** API version and object type.
- **metadata:** name, namespace, labels and annotations.
- **spec:** desired configuration; controllers reconcile actual state toward it.
- **Labels/selectors:** connect Deployments, Services, and other controllers to matching Pods.
- **Controllers:** watch objects and continually work toward the requested state.
- **Namespace:** logical scope for many namespaced resources; not a complete security boundary by itself.
- **RBAC:** grants API permissions to users, groups and ServiceAccounts.
- **NetworkPolicy:** Pod traffic policy; it only enforces if the CNI supports it.
- **PV/PVC:** cluster storage resource and namespaced storage request.
- **CRDs:** extend the Kubernetes API; a custom resource such as PrometheusRule cannot be created until its CRD/controller is installed.

## Validation

For the initial Nginx example:
```bash
kubectl get all -n k8s-demo
kubectl describe deployment nginx -n k8s-demo
kubectl get endpointslice -n k8s-demo
kubectl get events -n k8s-demo --sort-by=.lastTimestamp
```

Only delete objects you created for the exercise. Deleting the namespace removes all namespaced resources inside it; PVs may be cluster-scoped and need separate lifecycle handling.
