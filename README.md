# Kubernetes Installer

Requirements:

- one or more nodes running container-capable OS:
  - linux
  - systemd
  - network
- hardware requirements (maybe virtual), per node:
  - 2 CPU or more of compute
  - 2 GiB or more of memory
  - 20 GiB or more of storage

Components:

1) Container runtime [required]
   - containerd
2) Kubernetes requirements
   - cri
   - cni (bridge)
3) Kubernetes control plane
   - kubelet
   - kube-apiserver
   - etcd
   - coredns
4) Kubernetes pod network [optional]
   - cni (flannel)
5) Kubernetes worker(s) [optional]
   - kubelet
6) Kubernetes dashboard [optional]
   - dashboard
   - metrics-server

<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/>

