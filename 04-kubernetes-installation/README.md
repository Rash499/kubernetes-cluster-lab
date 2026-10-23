# 04 — Kubernetes Installation with kubeadm

## Concept
kubeadm bootstraps the cluster; kubelet manages node workloads; kubectl talks to the API server. The notes record Kubernetes v1.34.12 from the v1.34 APT repository.

## Install packages
```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor --yes -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

APT source:
```text
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /
```

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Initialize control plane (new cluster only)
```bash
sudo kubeadm init --apiserver-advertise-address=192.168.6.138 --pod-network-cidr=10.244.0.0/16
mkdir -p "$HOME/.kube"
sudo cp -i /etc/kubernetes/admin.conf "$HOME/.kube/config"
sudo chown "$(id -u):$(id -g)" "$HOME/.kube/config"
kubectl get nodes
```

Join workers using a fresh command generated on the control plane:
```bash
sudo kubeadm token create --print-join-command
```
Run the generated command on each worker; never commit the token.

## Troubleshooting
For APT signature errors, verify keyring/repository and system time. For NotReady, inspect node conditions, kubelet, containerd and CNI. For join failures, check TCP 6443 reachability, token expiry, CA hash and firewall. Do not rerun kubeadm init on a working cluster.