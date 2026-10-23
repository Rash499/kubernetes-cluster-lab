# 11 — Ansible Worker Automation

## Concept
Ansible runs idempotent tasks over SSH using inventory and playbooks. Earlier conversation records the plan to configure workers one by one; treat the full automation as in progress until a successful run is verified.

## Inventory example
Replace each masked address with the corresponding worker's actual LAN address in your private inventory:
```ini
[k8s_workers]
k8s-worker1 ansible_host=192.168.xxx.xxx
k8s-worker2 ansible_host=192.168.xxx.xxx

[k8s_workers:vars]
ansible_user=<ssh-user>
ansible_become=true
```

## Check connectivity
```bash
ansible-inventory -i inventory.ini --list
ansible k8s_workers -i inventory.ini -m ping
```

## Example preparation playbook
```yaml
---
- name: Prepare Kubernetes worker nodes
  hosts: k8s_workers
  become: true
  tasks:
    - name: Configure required modules
      ansible.builtin.copy:
        dest: /etc/modules-load.d/k8s.conf
        owner: root
        group: root
        mode: "0644"
        content: |
          overlay
          br_netfilter
    - name: Enable IPv4 forwarding
      ansible.posix.sysctl:
        name: net.ipv4.ip_forward
        value: "1"
        state: present
        reload: true
```

## Safe rollout
```bash
ansible-playbook -i inventory.ini playbooks/prepare-nodes.yml --syntax-check
ansible-playbook -i inventory.ini playbooks/prepare-nodes.yml --check --diff
ansible-playbook -i inventory.ini playbooks/prepare-nodes.yml --limit k8s-worker1
```
Install needed collections first. Test one worker, then expand. Never store passwords or private keys in Git.
