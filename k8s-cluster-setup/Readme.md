Pre-installation:
- Ansible should be installed in local machine.
- Ensure that the target machines are reachable via SSH.
- Set hosts in the Ansible inventory file inventory.ini.
- Set user in group_vars/all.yml (ansible_user field) if user is different than root. User should have sudo privileges.
- Ensure that Python 3.11 is installed on target machines. You can run `ansible -i inventory.ini all -m raw -a "zypper install -y python311"` for this.

Playbook execution:
- Run `ansible-playbook -i inventory.ini site.yml`

Post-installation:
- To add kubeconfig on your local machine run:
```
mkdir -p ~/.kube
scp root@<master IP>:/etc/kubernetes/admin.conf ~/.kube/config
chmod 600 ~/.kube/config
kubectl get nodes
```