# Sources and accuracy notes

This documentation consolidates available project conversations and the following Notion pages:

- [Kubernetes Setup](https://app.notion.com/p/3a7de57da8e98069b6cdc77525cda8af)
- [Kubernetes Setup — Topics 1–35](https://app.notion.com/p/3edde57da8e981af9a1eed90aa4cfb17)
- [Kubernetes Setup — Topics 36–52](https://app.notion.com/p/3edde57da8e981dda53fcc77327166f1)
- [Topic 51 — Kubernetes NetworkPolicy](https://app.notion.com/p/3f3de57da8e981b3b240e3840c2b8b93)
- [Topic 52 — NetworkPolicy Traffic Isolation Test](https://app.notion.com/p/3f3de57da8e98191b582ed3c915226dc)
- [Topic 60 — Kubernetes Services Deep Dive](https://app.notion.com/p/3edde57da8e981dfb833fa187a56eefd)
- [Topic 61 — Kubernetes Networking Deep Dive](https://app.notion.com/p/3edde57da8e981fb9d79e61c915226dc)

Historical values: Kubernetes v1.34.12, containerd 1.7.24, Pod CIDR 10.244.0.0/16, node addresses 192.168.6.138/.139/.140. Later notes describe a Flannel-to-Calico v3.33.0 transition. Verify the live cluster before relying on current versions or resource addresses.

Never publish credentials, private SSH keys, kubeconfig files, join tokens or unredacted Secret output.