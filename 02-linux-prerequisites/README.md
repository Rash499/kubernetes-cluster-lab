# 02 — Linux Prerequisites

## Concept
Kubernetes depends on appropriate kernel networking features and predictable node memory settings. The lab prepared swap, modules, sysctl and host connectivity.

## Commands
```bash
free -h
swapon --show
sudo swapoff -a
sudo modprobe overlay
sudo modprobe br_netfilter
lsmod | grep -E 'overlay|br_netfilter'
```

Persist modules in /etc/modules-load.d/k8s.conf:
```text
overlay
br_netfilter
```

Persist networking in /etc/sysctl.d/k8s.conf:
```text
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
```

Apply and verify:
```bash
sudo sysctl --system
sysctl net.ipv4.ip_forward
sysctl net.bridge.bridge-nf-call-iptables
grep -n swap /etc/fstab
```

## Troubleshooting
If bridge sysctls are missing, load br_netfilter. If swap returns after reboot, inspect /etc/fstab and swap-management services. If nodes cannot communicate, check IPs, routes, firewall and VMware networking.