# Kubernetes Cluster Configuration Cookbook

Based on the supplied **Kubernetes Cluster on VMware — Complete Lab Guide (Topics 1–61)**. This quick reference focuses on commands and configuration examples needed to reproduce the lab.

> **Public-repository safety:** LAN addresses are intentionally masked. Replace every 192.168.xxx.xxx placeholder with your actual VMware LAN address. Never commit kubeadm join tokens, kubeconfig files, private keys, passwords, or real Secret values.
>
> Lab facts recorded in the guide: Debian 13 (trixie), Kubernetes v1.34.12 via kubeadm, containerd 1.7.24, interface ens33, Pod CIDR 10.244.0.0/16. Flannel was used initially; the lab later migrated to Calico v3.33.0 for NetworkPolicy. Check the active CNI before changing networking.

## 1. Node inventory and hostnames

| Role | Hostname | Address |
|---|---|---|
| Control plane | k8s-master | 192.168.xxx.xxx |
| Worker 1 | k8s-worker1 | 192.168.xxx.xxx |
| Worker 2 | k8s-worker2 | 192.168.xxx.xxx |

Run the relevant hostname command on each VM:

```bash
# Control-plane VM
sudo hostnamectl set-hostname k8s-master

# Worker 1 VM
sudo hostnamectl set-hostname k8s-worker1

# Worker 2 VM
sudo hostnamectl set-hostname k8s-worker2
```

Check addressing and routing before changing network settings:

```bash
hostname
ip -br addr
ip route
cat /etc/resolv.conf
```

Add entries on all three nodes to /etc/hosts, replacing placeholders with real LAN addresses:

```text
192.168.xxx.xxx  k8s-master
192.168.xxx.xxx  k8s-worker1
192.168.xxx.xxx  k8s-worker2
```

Verify resolution and connectivity:

```bash
getent hosts k8s-master k8s-worker1 k8s-worker2
ping -c 3 k8s-worker1
ping -c 3 k8s-worker2
```

## 2. Prepare Linux on every node

### Disable swap

```bash
sudo swapoff -a
free -h
sudo nano /etc/fstab
```

Comment out the active swap entry in /etc/fstab by prefixing it with #, so swap remains disabled after reboot.

### Load kernel modules

```bash
sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/modules-load.d/k8s.conf <<'EOF'
overlay
br_netfilter
EOF

lsmod | grep -E 'overlay|br_netfilter'
```

### Configure sysctl networking

```bash
sudo tee /etc/sysctl.d/k8s.conf <<'EOF'
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
sysctl net.ipv4.ip_forward
```

Expected forwarding value: net.ipv4.ip_forward = 1.

## 3. Install and configure containerd on every node

```bash
sudo apt update
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo nano /etc/containerd/config.toml
```

In /etc/containerd/config.toml, set the runtime's systemd cgroup driver:

```toml
SystemdCgroup = true
```

Restart, enable and verify containerd:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
sudo systemctl status containerd
sudo containerd config dump | grep SystemdCgroup
```

## 4. Install Kubernetes packages on every node

The lab uses the official v1.34 APT repository:

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key |
  sudo gpg --dearmor --yes -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' |
  sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable kubelet

kubeadm version
kubelet --version
kubectl version --client
```

Do not bypass APT signature verification if a key error occurs. Recheck the keyring and repository configuration.

## 5. Initialize the control plane

**Run only on k8s-master.** Replace the advertised address with the real control-plane IP:

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.xxx.xxx \
  --pod-network-cidr=10.244.0.0/16
```

After successful initialization, configure kubectl for the administrative user:

```bash
mkdir -p "$HOME/.kube"
sudo cp -i /etc/kubernetes/admin.conf "$HOME/.kube/config"
sudo chown "$(id -u):$(id -g)" "$HOME/.kube/config"
kubectl get nodes
```

The control plane may remain NotReady until a CNI is installed and healthy.

### Join workers

Use the exact kubeadm join command printed by kubeadm init on each worker. Do not publish the token. If the token expires, generate a new command on the control plane:

```bash
kubeadm token create --print-join-command
```

## 6. CNI and cluster validation

The guide records Flannel as the initial CNI and a later migration to Calico v3.33.0. **Do not install a second CNI on top of an existing cluster.** Inspect the active setup and follow the migration notes before changing it.

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get daemonsets -A
kubectl get events -A --sort-by=.lastTimestamp
```

Troubleshooting checks:

```bash
sudo systemctl status containerd kubelet
sudo journalctl -u containerd --since "15 minutes ago" --no-pager
sudo journalctl -u kubelet --since "15 minutes ago" --no-pager
sudo crictl info
ip route
ip -br addr
```

If a CNI executable is missing, check the actual CNI binary path. The lab notes document a mismatch between /usr/lib/cni and /opt/cni/bin.

## 7. Workloads, Services and DNS

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
kubectl get svc,endpointslices -A
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl get events -A --sort-by=.lastTimestamp
```

A Service needs matching Pod labels. Inspect its selector and EndpointSlices when it has no backends. For DNS failures, inspect CoreDNS Pods and test service names from a temporary Pod.

## 8. ConfigMaps and Secrets

Example ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: "lab"
  LOG_LEVEL: "info"
```

Apply and inspect:

```bash
kubectl apply -f configmap.yaml
kubectl describe configmap app-config
```

Create a lab Secret interactively instead of tracking a file with real credentials:

```bash
kubectl create secret generic app-credentials \
  --from-literal=username='<lab-user>' \
  --from-literal=password
```

Base64 is encoding, not encryption. Never commit real Secret values or command histories containing credentials.

## 9. Storage, Ingress and Helm checks

```bash
kubectl get pv,pvc -A
kubectl describe pvc <claim-name> -n <namespace>
kubectl get ingress -A
kubectl describe ingress <ingress-name> -n <namespace>
helm list -A
helm history <release-name> -n <namespace>
helm get values <release-name> -n <namespace>
```

Storage manifests may require a StorageClass or prepared host path. Ingress manifests require a compatible controller. Read individual manifest comments before applying.

## 10. Monitoring, resources and autoscaling

```bash
kubectl top nodes
kubectl top pods -A
kubectl get hpa -A
kubectl describe hpa <hpa-name> -n <namespace>
kubectl get prometheusrules -A
```

HPA requires a working Metrics Server and resource requests for relevant container metrics. PrometheusRule resources require the Prometheus Operator CRDs from the monitoring stack.

## 11. RBAC and NetworkPolicy

```bash
kubectl get roles,rolebindings -A
kubectl get clusterroles,clusterrolebindings
kubectl auth can-i get pods -n <namespace> --as=<test-user>
kubectl get networkpolicy -A
kubectl describe networkpolicy <policy-name> -n <namespace>
```

NetworkPolicy enforcement depends on a CNI that supports it. Test allowed and denied traffic using the intended namespaces and labels; the existence of a policy object alone does not prove enforcement.

## 12. Safe-use checklist

- Replace every masked LAN address before executing commands.
- Verify hostname, interface, route, gateway and DNS on your own VMware network.
- Confirm which CNI is active; Flannel-to-Calico migration is a separate operation.
- Do not apply every manifest at once. Check prerequisites such as CRDs, controllers, StorageClasses, node labels and local paths.
- Never commit kubeconfig files, join tokens, private keys, passwords or live Secret data.
- Treat examples as lab templates, not proof that every resource remains deployed.

## Source and scope

Derived from the user-supplied *Kubernetes Cluster on VMware — Complete Lab Guide, Topics 1–61* PDF. This file organizes central configuration steps into a repository-friendly quick reference. The existing numbered directories and manifests folder provide focused topic documentation and YAML examples.
