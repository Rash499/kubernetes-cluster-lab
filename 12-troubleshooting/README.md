# 12 — Troubleshooting Journal

| Symptom | Checks | Lab lesson |
|---|---|---|
| APT signature/repository error | apt update; inspect source and keyring | Correct Kubernetes repository/keyring |
| CRI RuntimeService unavailable | ctr plugins ls; journalctl -u containerd | Verify CRI plugin and containerd config |
| Pod stuck ContainerCreating | describe Pod; inspect CNI DaemonSet and logs | CNI binary path mismatch was found |
| CoreDNS not ready / DNS fails | CoreDNS logs, Service, EndpointSlice, CNI | Correct CNI config and recreate affected Pods |
| ping missing in container | inspect image tools | Use diagnostic image with network utilities |
| echo workload RunContainerError | describe Pod; logs --previous | Replaced failing test image with Python HTTP server |
| kubectl connects to localhost:8080 | kubectl config view --minify; check KUBECONFIG | Select the intended kubeconfig |
| SSH Permission denied | ssh -vvv; username, authorized_keys and permissions | Correct SSH identity/key setup |

## Diagnostic commands
```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get events -A --sort-by=.lastTimestamp
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
sudo journalctl -u kubelet -b --no-pager -n 100
sudo journalctl -u containerd -b --no-pager -n 100
```

## Method
Observe status → describe resource/events → inspect logs → verify node/runtime/CNI → test Pod IP → Service IP → DNS → change one thing → retest and record.