# Ansible

### Example running playbook
- running basic playbook
```
ansible-playbook kubernetes_enable-groups.yaml -i hosts/kubernetes
```

- running playbook with environment variable
```
ansible-playbook kubernetes_k3s-worker-join.yaml -i hosts/kubernetes --extra-vars "kubernetes_master_url=https://192.168.100.10:6443” --extra-vars "kubernetes_token=token”
```

