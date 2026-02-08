# Kubernetes HA Cluster Setup

Deploys a highly available Kubernetes cluster with multiple control plane nodes using HAProxy and keepalived on Ubuntu.

## Architecture

```
                 VIP (floating IP via keepalived)
                          :8443
                            |
         +------------------+------------------+
         |                  |                  |
    CP Node 1          CP Node 2          CP Node 3
    haproxy:8443       haproxy:8443       haproxy:8443
    keepalived         keepalived         keepalived
    apiserver:6443     apiserver:6443     apiserver:6443
    etcd               etcd               etcd
```

- **keepalived** manages a floating VIP across control plane nodes via VRRP
- **HAProxy** load balances kube-apiserver traffic (port 8443 -> 6443) across all control plane nodes
- The first master (`masters[0]`) initializes the cluster; additional masters join with `--control-plane`
- Workers connect to the API server through the VIP

## Pre-installation

- Ansible should be installed on the local machine:
```
sudo apt update && sudo apt install -y python3 python3-pip pipx
pipx ensurepath
pipx install --include-deps ansible
```
- Ensure that the target machines are reachable via SSH:
```
ssh-copy-id <user>@<target IP>
```
- Ensure that Python 3 is installed on target machines. You can run `ansible -i inventory.ini all -m raw -a "apt install -y python3"` for this.

## Configuration

### Inventory

Set hosts in `inventory.ini`. The first host under `[masters]` will initialize the cluster:

```ini
[masters]
10.2.127.5    # First master - initializes the cluster
10.2.127.6
10.2.127.7

[workers]
10.2.127.8
10.2.127.9
```

### Variables

Edit `group_vars/all.yaml`:

| Variable | Description | Default |
|----------|-------------|---------|
| `ansible_user` | SSH user with sudo privileges | `root` |
| `kubernetes_version` | Kubernetes version to install | `1.32.0` |
| `ha_cluster_vip` | Floating virtual IP for the control plane | `10.2.127.100` |
| `ha_cluster_vip_mask` | CIDR prefix for the VIP | `24` |
| `haproxy_port` | HAProxy frontend port (must differ from 6443) | `8443` |
| `keepalived_vrrp_interface` | Network interface for VRRP traffic | `ens33` |
| `keepalived_virtual_router_id` | VRRP router ID (unique per L2 segment) | `60` |
| `keepalived_auth_pass` | VRRP authentication password | `1111` |

## Playbook execution

```
ansible-playbook -i inventory.ini site.yml
```

The playbook executes in this order:
1. Prepare all nodes (system config, containerd, kubernetes packages)
2. Deploy HAProxy and keepalived on all control plane nodes
3. Initialize the first control plane node
4. Deploy Calico CNI
5. Join additional control plane nodes (one at a time)
6. Join worker nodes
7. Verify cluster status

## Post-installation

To add kubeconfig on your local machine (use the VIP address):
```
mkdir -p ~/.kube
scp root@<VIP or any master IP>:/etc/kubernetes/admin.conf ~/.kube/config
chmod 600 ~/.kube/config
kubectl get nodes
```

To verify the cluster:
```
ansible-playbook -i inventory.ini verify.yml
```
