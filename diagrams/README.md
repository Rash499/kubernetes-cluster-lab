# Architecture Diagrams

These Mermaid diagrams describe the recorded lab design. The CNI changed over time, so Flannel is historical and later notes record Calico v3.33.0. Confirm the active cluster state before using a diagram as an as-built inventory. LAN node addresses are intentionally masked.

## Full cluster

```mermaid
flowchart TB
  Host["Windows 11"] --> VMware["VMware"]
  VMware --> CP["k8s-master · 192.168.xxx.xxx"]
  VMware --> W1["k8s-worker1 · 192.168.xxx.xxx"]
  VMware --> W2["k8s-worker2 · 192.168.xxx.xxx"]
  CP --> API["kube-apiserver :6443"]
  API --> ETCD["etcd"]
  API --> SCH["scheduler"]
  API --> CCM["controller manager"]
  W1 --> K1["kubelet + containerd"]
  W2 --> K2["kubelet + containerd"]
  K1 --> CNI["CNI: Flannel historically / Calico later"]
  K2 --> CNI
  CNI --> Pods["Pod CIDR 10.244.0.0/16"]
  Pods --> Svc["Services + EndpointSlices"]
  Pods --> DNS["CoreDNS"]
  API --> Security["Pod Security Admission / RBAC"]
  Pods --> Monitor["Prometheus / Grafana"]
  Ansible["Ansible controller"] -. "SSH automation workflow" .-> W1
  Ansible -. "SSH automation workflow" .-> W2
```

## Service traffic and DNS

```mermaid
flowchart LR
  Client["Client Pod"] --> DNS["kube-dns Service"] --> CoreDNS["CoreDNS"]
  Client --> SVC["nginx-service ClusterIP"] --> EPS["EndpointSlice"]
  EPS --> P1["Nginx Pod A"]
  EPS --> P2["Nginx Pod B"]
  EPS --> P3["Nginx Pod C"]
```

## NodePort / kube-proxy packet path

```mermaid
flowchart LR
  Client["Client"] --> Node["Node IP : NodePort"]
  Node --> NP["KUBE-NODEPORTS"]
  NP --> EXT["KUBE-EXT-*"]
  EXT --> SVC["KUBE-SVC-*"]
  SVC --> SEP["KUBE-SEP-*"]
  SEP --> DNAT["DNAT to Pod IP:targetPort"]
```

This shows the iptables-based path observed in the lab. Chain names and dataplane differ across Kubernetes configurations.

## LoadBalancer with MetalLB

```mermaid
flowchart TB
  Client["LAN client"] --> VIP["External VIP from a reserved private pool"]
  VIP --> L2["MetalLB speaker / L2 advertisement"]
  L2 --> Node["Node receiving traffic"]
  Node --> Svc["Kubernetes Service dataplane"]
  Svc --> EPS["EndpointSlice"]
  EPS --> Pods["Ready backend Pods"]
```

## Monitoring and alerting

```mermaid
flowchart LR
  Nodes["Nodes / Pods"] --> Exporters["Node Exporter / kube-state-metrics"]
  Exporters --> Prom["Prometheus"]
  Prom --> Grafana["Grafana dashboards"]
  Prom --> Rules["PromQL alert rules"]
  Rules --> AM["Alertmanager"]
```

## Persistent storage

```mermaid
flowchart TB
  Pod["Test Pod"] --> Mount["/data mount"]
  Mount --> PVC["PVC app-pvc"]
  PVC --> PV["PV k8s-local-pv"]
  PV --> HostPath["hostPath on worker1"]
```

## NetworkPolicy logic

```mermaid
flowchart TB
  Allowed["Client Pod with matching label"] --> Policy{"Ingress policy permits TCP/80?"}
  Denied["Client Pod without label"] --> Policy
  Policy -->|yes| Server["Selected server Pod"]
  Policy -->|no, when CNI enforces policy| Drop["Traffic denied"]
```

## Ansible workflow

```mermaid
flowchart LR
  A["Ansible controller"] --> SSH["Inventory + SSH"]
  SSH --> W1["worker1"]
  SSH --> W2["worker2"]
  W1 --> Prep["OS prerequisites / runtime / packages"]
  W2 --> Prep
  Prep --> Verify["Verify Kubernetes node health"]
```
