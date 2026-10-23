# 12 — Troubleshooting Journal and Runbooks

Use a repeatable sequence: observe the symptom, inspect object events, read logs, verify dependencies, change one thing, and retest. Avoid deleting resources before collecting useful evidence.

## Recorded incidents and diagnostic paths

| Symptom | Check first | Lab lesson / likely cause |
|---|---|---|
| APT signature or repository error | `apt update`, repo file, keyring path and permissions | Correct the Kubernetes APT source/keyring before installing packages |
| kubeadm says `unknown service runtime.v1.RuntimeService` | `sudo ctr plugins ls | grep cri`; `journalctl -u containerd` | CRI plugin/configuration must be healthy |
| Pod stuck in `ContainerCreating` | `kubectl describe pod`; CNI DaemonSet/logs; node runtime logs | Lab found Flannel binary in `/opt/cni/bin` while containerd searched `/usr/lib/cni` |
| CoreDNS stuck / Service DNS fails | CoreDNS events/logs, kube-dns Service/EndpointSlice, CNI | CNI path issue also affected CoreDNS; DNS depends on functioning Pod networking |
| Test image enters `RunContainerError` / `CrashLoopBackOff` | `describe pod`, `logs`, `logs --previous`, exit code | The first echo image failed; replacing it with a simple Python HTTP server allowed Service testing |
| Service exists but traffic fails | selectors, Pod labels/readiness, EndpointSlices, targetPort, app listener | A Service cannot route successfully without eligible ready backends |
| Ingress returns 404 | IngressClass, host/path, HTTP Host header, controller logs | Ingress resource is only a rule; a controller must interpret it |
| Ingress returns 502/503 | Service endpoints, readiness, targetPort, controller logs | Controller can be healthy while its backend is unavailable |
| NodePort unreachable | node IP/port, firewall, route, kube-proxy rules and endpoint location | With `externalTrafficPolicy: Cluster`, a node may forward to a Pod on another node |
| LoadBalancer IP remains pending/unreachable | MetalLB Pods, IPAddressPool, L2Advertisement, ARP/neighbour table, IP conflicts | VMware needs a LoadBalancer implementation; confirm the address pool is unused |
| NetworkPolicy does not block traffic | active CNI and policy support, selectors, direction and test endpoints | API acceptance alone does not guarantee CNI enforcement |
| HPA target shows unknown | Metrics Server, APIService, Pod requests and HPA events | Prometheus metrics are separate from the Kubernetes Metrics API |
| `kubectl top` says Metrics API unavailable | `kubectl get apiservice v1beta1.metrics.k8s.io`; Metrics Server logs | Fix Metrics Server/API integration, not Grafana |
| kubectl tries localhost:8080 | `kubectl config view --minify`, `echo $KUBECONFIG` | User context lacked a kubeconfig pointing to the API server |
| SSH says Permission denied | `ssh -vvv`, username, public key, `authorized_keys` and permissions | Check the intended key/user rather than repeatedly guessing passwords |
| Pod is OOMKilled | `describe pod`, previous logs, resource limit and memory use | Memory limit breach can terminate a container; CPU pressure behaves differently |
| Pod cannot schedule on a tainted node | node taints, toleration key/value/effect, nodeSelector/affinity | Toleration permits placement; selector/affinity chooses a node |

## First-pass cluster commands

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get svc,endpointslice -A
kubectl get events -A --sort-by=.lastTimestamp
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous
kubectl get pods -n kube-system -o wide
sudo journalctl -u kubelet -b --no-pager -n 100
sudo journalctl -u containerd -b --no-pager -n 100
sudo ctr plugins ls | grep cri
```

## Networking isolation sequence

1. Confirm the Pod is Ready and record its node/IP: `kubectl get pod -o wide`.
2. Test the application directly on its Pod IP and target port.
3. Inspect the Service selector and EndpointSlice.
4. Test the Service ClusterIP.
5. Test the Service DNS name.
6. For NodePort/LoadBalancer, test node reachability, external address, firewall and dataplane rules.
7. For NetworkPolicy, compare allowed and denied clients in a dedicated test namespace and verify CNI enforcement.

## CNI-specific checks

```bash
kubectl get daemonsets -A
kubectl get pods -A | grep -Ei 'calico|flannel'
kubectl get nodes -o wide
kubectl get pods -A -o wide
sudo find /etc/cni/net.d /opt/cni/bin /usr/lib/cni -maxdepth 2 -type f 2>/dev/null
sudo journalctl -u containerd -b --no-pager -n 100
```

The initial cluster used Flannel. Later Notion notes describe Calico v3.33.0. Do not apply a fix intended for one CNI until you confirm which plugin is active.

## Safe troubleshooting habits

- Capture `describe`, events and logs before deleting or restarting workloads.
- Avoid copying cluster-admin kubeconfig to worker nodes for convenience; use least-privilege access.
- Never paste real Secret values, join tokens or private key material into public issues.
- Treat generated IPs, Service names, endpoint IPs and iptables chain suffixes as observations, not permanent constants.
- After a change, repeat the same test and record the before/after result.
