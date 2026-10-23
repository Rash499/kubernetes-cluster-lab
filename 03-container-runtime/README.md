# 03 — containerd Runtime

## Concept
Kubelet uses the Container Runtime Interface (CRI) to ask containerd to pull images and run containers. The notes record containerd 1.7.24 and SystemdCgroup=true.

## Implementation
```bash
sudo apt update
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
sudo systemctl enable --now containerd
sudo systemctl status containerd --no-pager
containerd --version
```

Ensure the relevant runtime config uses:
```toml
SystemdCgroup = true
```

The earlier CNI setup needed these paths:
```toml
[plugins."io.containerd.grpc.v1.cri".cni]
  bin_dir = "/opt/cni/bin"
  conf_dir = "/etc/cni/net.d"
```

```bash
sudo systemctl restart containerd
sudo ctr plugins ls | grep cri
sudo journalctl -u containerd -b --no-pager -n 100
```

## Troubleshooting
For `unknown service runtime.v1.RuntimeService`, inspect CRI plugin health and containerd logs. For `failed to find plugin ... flannel`, verify actual CNI binary location and containerd's bin_dir. Back up config before regenerating it.