# Build a Kubernetes Cluster using `kubeadm` via Ansible

This repository contains the Ansible playbook that I use to install [Kubernetes](https://kubernetes.io) on my [Raspberry Pi cluster](https://heathharrelson.github.io/posts/raspberry-pi-cluster-kubeadm/). This playbook installs a single master production-like configuration using [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/). [Containerd](https://containerd.io) is used as the container runtime.

I built this using [Ubuntu 20.04 for arm64](https://ubuntu.com/download/raspberry-pi), although it should be relatively easy to get working with other Debian-based distributions.

This playbook is loosely based on the [k3s Ansible playbook](https://github.com/k3s-io/k3s-ansible) and incorporates parts of that playbook's Raspberry Pi role.

## Usage

The inventory provided in `inventory/hosts.ini` is specific to my configuration and will need to be edited or replaced.

```ini
[master]
192.168.4.42

[node]
192.168.4.43
192.168.4.44
192.168.4.45

[all:children]
master
node

[all:vars]
ansible_user=k8s
ansible_ssh_private_key=~/.ssh/rpi_cluster_rsa
```

Install dependencies from Ansible Galaxy.

```
$ ansible-galaxy install -r requirements.yml
```

Run the playbook to set up the cluster.

```
$ ansible-playbook main.yml
```

## Customization

### Kubernetes

To configure the versions of containerd or the Kubernetes cluster, edit `vars/main.yml`.

```yaml
---
# geerlingguy.swap options
swap_file_state: absent

# geerlingguy.containerd options
containerd_package: containerd.io=1.4.3-1
docker_apt_arch: arm64

# geerlingguy.kubernetes options
kubernetes_version: '1.20'
kubernetes_packages:
  - name: kubeadm=1.20.4-00
    state: present
  - name: kubelet=1.20.4-00
    state: present
  - name: kubectl=1.20.4-00
    state: present
kubernetes_allow_pods_on_master: false
kubernetes_enable_web_ui: false
kubernetes_apiserver_advertise_address: '10.1.1.1'
```

See the documentation for the [geerlingguy.containerd](https://github.com/geerlingguy/ansible-role-containerd) and [geerlingguy.kubernetes](https://github.com/geerlingguy/ansible-role-kubernetes) roles for the available options.
