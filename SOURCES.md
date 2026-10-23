# Sources and accuracy notes

This repository was audited against the connected Notion Kubernetes lab pages available during this update.

## Main lab record

- [Kubernetes Setup](https://app.notion.com/p/3a7de57da8e98069b6cdc77525cda8af)
- [Kubernetes Setup — Topics 1–35](https://app.notion.com/p/3edde57da8e981af9a1eed90aa4cfb17)
- [Kubernetes Setup — Topics 36–52](https://app.notion.com/p/3edde57da8e981dda53fcc77327166f1)

## Deep dives and later topics

- [Topic 37 — Kubernetes Ingress](https://app.notion.com/p/3f3de57da8e981c68976e39154964f68)
- [Topic 39 — Helm](https://app.notion.com/p/3f3de57da8e981b991d2f56158996f5d)
- [Topic 44 — Monitoring with Prometheus & Grafana](https://app.notion.com/p/3f3de57da8e98102a3f3feeacd7a709f)
- [Topic 45 — Prometheus & PromQL](https://app.notion.com/p/3f3de57da8e98122a02afffdf9051c14)
- [Topic 46 — Alerts & Alertmanager](https://app.notion.com/p/3f3de57da8e981e3ab18cad6e01b9752)
- [Topic 47 — Resource Requests, Limits & QoS](https://app.notion.com/p/3f3de57da8e981ef8acaf9926dffbcfb)
- [Topic 48 — Horizontal Pod Autoscaler](https://app.notion.com/p/3f3de57da8e981f591fed651132e1305)
- [Topic 49 — RBAC](https://app.notion.com/p/3f3de57da8e98138b572c76e08235d3a)
- [Topic 50 — ClusterRole and ClusterRoleBinding](https://app.notion.com/p/3f3de57da8e9814ea317d9baa4d6ec95)
- [Topic 51 — NetworkPolicy](https://app.notion.com/p/3f3de57da8e981b3b240e3840c2b8b93)
- [Topic 52 — NetworkPolicy Traffic Isolation Test](https://app.notion.com/p/3f3de57da8e98191b582ed3c719bc5d6)
- [Topic 54 — Taints and Tolerations](https://app.notion.com/p/3f3de57da8e9812b861afc07b52b71ca)
- [Topic 56 — StatefulSets](https://app.notion.com/p/3f3de57da8e981a38c37dfd43a6be95b)
- [Topic 57 — DaemonSets](https://app.notion.com/p/3f3de57da8e9817aa4f3f162965d1f82)
- [Topic 58 — Jobs and CronJobs](https://app.notion.com/p/3f3de57da8e981c1b347dfe96df55365)
- [Topic 59 — Probes](https://app.notion.com/p/3f3de57da8e98195a6f6df8b7abfb65c)
- [Topic 60 — Services Deep Dive](https://app.notion.com/p/3f3de57da8e981dfb833fa187a56eefd)
- [Topic 61 — Networking Deep Dive](https://app.notion.com/p/3f3de57da8e981fb9d79e61c915226dc)

## Accuracy and safety notes

- The Notion setup says node addresses are DHCP-assigned; they were not configured as static IPs in that record. Private LAN addresses have been masked in this public repository as `192.168.xxx.xxx`.
- The early CNI was Flannel. Later notes describe a migration to Calico v3.33.0. Verify the current active CNI before changing anything.
- The cluster versions recorded were Kubernetes v1.34.12 and containerd 1.7.24.
- The Service deep dive records MetalLB v0.15.3 and an example external address range. The actual addresses are intentionally masked; choose and verify a valid range reserved for the lab and excluded from DHCP before applying.
- The HPA notes identify Metrics Server as the source for the Kubernetes Metrics API used by `kubectl top` and standard resource-metric HPA; Prometheus is a separate monitoring path.
- YAML and Helm examples in this repository are learning templates, not proof that every object remains deployed or that every example is production-ready. Masked IP placeholders are not valid configuration values and must be replaced locally before running.
- Ansible worker automation is documented as a workflow to verify rather than a completed result.
- Never publish credentials, private SSH keys, kubeconfig files, join tokens or unredacted Secret output.
