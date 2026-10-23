# 13 — Validation and Interview Notes

## Validation checklist
```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get deployments,services -A
kubectl get endpointslice -A
kubectl get pv,pvc -A
kubectl get ingress -A
kubectl get networkpolicy -A
kubectl get roles,rolebindings -A
kubectl get clusterroles,clusterrolebindings
```

Historical resources may have been deleted; verify names before using follow-up commands.

## One-minute interview summary
“I built a multi-node Kubernetes lab on three Debian 13 VMware VMs using kubeadm and containerd. I prepared Linux prerequisites, configured kernel modules, packet forwarding and systemd cgroups, installed kubeadm/kubelet/kubectl, initialized the control plane and joined two workers. I configured CNI networking, initially Flannel and later documented a migration to Calico. I deployed Nginx and a Python HTTP test workload, exposed them with Services, and tested EndpointSlices, CoreDNS service discovery and cross-node Pod connectivity. I also practiced ConfigMaps, Secrets, PV/PVC, Ingress, Pod Security Admission with the Restricted profile, RBAC and NetworkPolicy. I troubleshot CRI, CNI paths, DNS and workload startup, and planned Ansible automation for repeatable worker preparation.”

Only describe Ansible/RBAC/Ingress details as completed if you have verified implementation evidence.

## Topics to prepare
Control-plane components; node components; Pod vs node vs Service CIDRs; Deployment vs Pod vs Service; EndpointSlices; CoreDNS; PV vs PVC; ConfigMap vs Secret; Pod Security Admission; RBAC least privilege; NetworkPolicy enforcement; and Ansible idempotence/check mode.