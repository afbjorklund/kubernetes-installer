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

Packages:
```
768K    cni-plugin-flannel-1.1.0-amd64.txz
23M     cni-plugins-1.1.1-amd64.txz
30M     containerd-1.6.6-amd64.txz
12M     crictl-1.24.2-amd64.txz
8.8M    kubeadm-1.25.0-amd64.txz
9.1M    kubectl-1.25.0-amd64.txz
19M     kubelet-1.25.0-amd64.txz
2.6M    runc-1.1.3-amd64.txz
104M    total
```

Images:
```
142M    kubernetes-1.25.0-amd64.txz
14M     flannel-0.19.2-amd64.txz
89M     dashboard-2.6.1-amd64.txz
```

