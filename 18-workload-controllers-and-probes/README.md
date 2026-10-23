# 18 — Workload Controllers, Probes and Scheduling

This guide captures additional topics recorded in Notion beyond basic Deployments.

## Controller comparison

| Resource | Intended use | Identity / behavior |
|---|---|---|
| Deployment | Stateless replicated apps | Manages ReplicaSets and rolling updates |
| StatefulSet | Stateful apps | Stable ordinal Pod identity and per-replica storage claims |
| DaemonSet | Node-level agents | One Pod on each eligible node |
| Job | Finite task | Retries until the configured completion outcome |
| CronJob | Scheduled tasks | Creates Jobs on a schedule |
| HPA | Replica autoscaling | Adjusts scalable workload replicas from metrics |

## Probes

- **Readiness:** determines whether a Pod should receive Service traffic.
- **Liveness:** detects a container that should be restarted.
- **Startup:** protects slow-starting applications by delaying liveness/readiness checks until startup succeeds.

A failed readiness probe removes a Pod from ready Service endpoints; a failed liveness probe can restart the container. Bad liveness probes can create restart loops, so use them for detecting unrecoverable states, not every temporary dependency failure.

## Taints, tolerations and node selection

A node taint repels Pods unless they have a matching toleration. Effects include `NoSchedule`, `PreferNoSchedule` and `NoExecute`. A toleration only permits placement; it does not select the node. Use `nodeSelector` or node affinity when a workload must target a node.

Notion records applying a special `workload=special:NoSchedule` taint to worker1, testing Pods with/without tolerations, combining a toleration with nodeSelector, and examining all three effects. Remove test taints when done.

## Example commands

```bash
kubectl get statefulsets,daemonsets,jobs,cronjobs -A
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl get events --sort-by=.lastTimestamp
kubectl rollout status daemonset/node-agent
kubectl rollout history daemonset/node-agent
kubectl scale statefulset web --replicas=2 -n statefulset-demo
kubectl describe hpa -A
```

## YAML examples

- [probes-demo.yaml](../manifests/probes-demo.yaml)
- [statefulset-demo.yaml](../manifests/statefulset-demo.yaml)
- [daemonset-demo.yaml](../manifests/daemonset-demo.yaml)
- [job-cronjob-demo.yaml](../manifests/job-cronjob-demo.yaml)
- [toleration-demo.yaml](../manifests/toleration-demo.yaml)

These are representative teaching examples. Review image compatibility, namespace, node labels/taints and cleanup impact before applying.

## Troubleshooting

- **Probe failures:** inspect Pod events, logs and probe path/port, startup delays, and whether the application listens on the configured interface.
- **StatefulSet PVC Pending:** check StorageClass, matching PV capacity/access mode and node affinity; stable identity alone does not provide storage.
- **DaemonSet missing from a node:** inspect node selectors, taints/tolerations and scheduling events.
- **Job retries:** inspect `kubectl describe job`, Pod exit codes and logs; use `backoffLimit` intentionally.
- **CronJob did not run:** check schedule/time zone behavior, suspension, concurrency policy and Job history.
- **Pod does not land on the tainted node:** ensure toleration key/value/effect match and use a node selector/affinity if placement is required.
