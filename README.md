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
30M     containerd-1.6.8-amd64.txz
12M     crictl-1.24.2-amd64.txz
8.8M    kubeadm-1.25.0-amd64.txz
9.1M    kubectl-1.25.0-amd64.txz
19M     kubelet-1.25.0-amd64.txz
2.6M    runc-1.1.4-amd64.txz
104M    total
```

Images:
```
142M    kubernetes-1.25.0-amd64.txz
14M     flannel-0.19.2-amd64.txz
89M     dashboard-2.6.1-amd64.txz
```

## About CPU and Memory (RAM)

In Kubernetes, 1 "core" actually means 1 vCPU or 1 thread

Normally it is the same as seen in `nproc` or `loadavg`.

Example output from `lscpu`, from a quad-core computer:

```
CPU(s):                          8
On-line CPU(s) list:             0-7
Thread(s) per core:              2
Core(s) per socket:              4
Socket(s):                       1
```

Each core is 1000 millicores, and each GiB is 1024 MiB.

Not all memory is seen in _total_, and even less _available_.

Example output from `free`, from a machine with 4 GiB:

```console
$ free -m
               total        used        free      shared  buff/cache   available
Mem:            3927         132        3348           3         445        3577
Swap:              0           0           0
```

## Installing a container runtime

```
CONTAINERD_VERSION=v1.6.8
RUNC_VERSION=v1.1.4
```

### containerd

```
/usr/local/bin/containerd

/usr/local/lib/systemd/containerd.service
```

Low-level client:
`/usr/local/bin/ctr`

Runtime dependency:
`/usr/local/sbin/runc`

```shell
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```

Verify that containerd is running and that cri plugin is enabled:

```console
$ sudo ctr version
Client:
  Version:  v1.6.8
  Revision: 9cd3357b7fd7218e4aec3eae239db1f68a5a6ec6
  Go version: go1.17.13

Server:
  Version:  v1.6.8
  Revision: 9cd3357b7fd7218e4aec3eae239db1f68a5a6ec6
  UUID: d40463b5-1c03-4cc5-b836-0936f36047c2
$ sudo ctr plugins ls
TYPE                                  ID                       PLATFORMS      STATUS
io.containerd.content.v1              content                  -              ok
...
io.containerd.grpc.v1                 cri                      linux/amd64    ok
```

### CRI tools

```
/usr/local/bin/crictl
```

Configure runtime:
`/etc/crictl.yaml`

Verify that the container runtime is configured and that cri is up:

```
$ sudo crictl version
Version:  0.1.0
RuntimeName:  containerd
RuntimeVersion:  v1.6.8
RuntimeApiVersion:  v1
```

### CNI plugins

```
/opt/cni/bin/bridge

/opt/cni/bin/*
```

Configure networking:
`/etc/cni/net.d/10-containerd-net.conflist`

## Installing kubeadm, kubelet and kubectl

https://dl.k8s.io/release/stable.txt

```
KUBERNETES_VERSION=v1.25.0
RELEASE_VERSION=v0.4.0
```

### kubeadm

```
/usr/local/bin/kubeadm

/etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

### kubelet

```
/usr/local/bin/kubelet

/usr/local/lib/systemd/kubelet.service
```

### kubectl

```
/usr/local/bin/kubectl

/etc/kubernetes/admin.conf
```

## Preparing the required container images

### Download all images from the registry

```
kubeadm config images list
```

```
registry.k8s.io/kube-apiserver:v1.25.0
registry.k8s.io/kube-controller-manager:v1.25.0
registry.k8s.io/kube-scheduler:v1.25.0
registry.k8s.io/kube-proxy:v1.25.0
registry.k8s.io/pause:3.8
registry.k8s.io/etcd:3.5.4-0
registry.k8s.io/coredns/coredns:v1.9.3
```

```
kubeadm config images pull
```

### Uncompressing images before recompressing

```shell
kubeadm config images list --kubernetes-version=v1.25.0 > images.txt
xargs -I % sudo ctr -n k8s.io image convert --uncompress % % < images.txt
```

### Saving all images to a compressed archive

```shell
xargs sudo ctr -n k8s.io images export - < images.txt > images.tar
pigz < images.tar > images.tgz
pixz < images.tar > images.txz
```

```
203M    images-1.25.0-amd64.tgz
142M    images-1.25.0-amd64.txz
```

### Loading all images from a compressed archive

```shell
cat images.txt
zcat images.tgz > images.tar
xzcat images.txz > images.tar
sudo ctr -n k8s.io images import - < images.tar
```

```
unpacking registry.k8s.io/kube-apiserver:v1.25.0 (sha256:aa556e212aaf21f935c369b292fbd03c9b75f3506c6332dd0368eea486bfce31)...done
unpacking registry.k8s.io/kube-controller-manager:v1.25.0 (sha256:f6a0d0c2459faa2b9d77d176d75c2869a2cab68ea897f307274642e4b769b355)...done
unpacking registry.k8s.io/kube-scheduler:v1.25.0 (sha256:cf920556727b3ac7028ff1cd855b01be830af3fcab6f8967a08a41a509c9e827)...done
unpacking registry.k8s.io/kube-proxy:v1.25.0 (sha256:e22d653f5804294aafdd1eb1d4caedc68fbde35acc4ecee28757d0f33718d8e4)...done
unpacking registry.k8s.io/pause:3.8 (sha256:7cdf65a038c5552cbfdfb0f963ecfba5f0cd6579b9048eaa796de3d32760048c)...done
unpacking registry.k8s.io/etcd:3.5.4-0 (sha256:70effad8f559facab9a3f8315945c337b6dc62f17e5dbd41c8338c564cf36304)...done
unpacking registry.k8s.io/coredns/coredns:v1.9.3 (sha256:26cb5368454cfa8ad8a0b982607bd154c15574f3722994a20eac3249eebf3c50)...done
```

## Configuring node system

### Disable swap

`sudo swapoff -a`

### Disable selinux

`sudo setenforce Permissive`

### Disable firewall

`sudo systemctl stop firewalld`

## Configuring container runtime

### Install and configure prerequisites

`lsmod | grep br_netfilter`

```console
$ cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```

`sysctl net.bridge.bridge-nf-call-iptables`

```console
$ cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
$ sudo sysctl --system
```

### Configuring the systemd cgroup driver

/etc/containerd/config.toml
```toml
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

Otherwise the cluster will crash after five minutes.

```console
$ containerd config default | grep SystemdCgroup
            SystemdCgroup = false
```

Note: `systemd_cgroup` is for `io.containerd.runtime.v1.linux` runtime!

Note: `SystemdCgroup` is for `runtime_type = "io.containerd.runc.v2"`.

### Overriding the sandbox (pause) image

/etc/containerd/config.toml
```toml
[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "registry.k8s.io/pause:3.8"
```

Otherwise you will end up with multiple "pause" images.

```console
$ containerd config default | grep sandbox_image
    sandbox_image = "k8s.gcr.io/pause:3.6"
```

Note: `k8s.gcr.io` was moved to `registry.k8s.io` with k8s 1.25

Note: Currently the old registry is redirecting to the new one.

## Initializing your control-plane node

`kubeadm init`

To choose a specific address or port for apiserver, use the flags:

`--apiserver-advertise-address string`
`--apiserver-bind-port int32` (6443)

When the initialization is complete, the following will be shown:

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

To show information about all the current nodes in the cluster:

```
kubectl get nodes --output=wide
```

### Control plane node scheduling

By default, your cluster will **not** schedule any Pods on the control plane nodes.

To remove this taint and schedule non-system pods everywhere:

```
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

This is useful for instance when not having any separate worker nodes available.

## Joining your (worker) nodes

### Installing a Pod network add-on

When using "bridge" CNI, only a single node is possible.

To run more than one node, one can use the "flannel" CNI.

```
FLANNEL_VERSION=v0.19.2
FLANNEL_CNI_PLUGIN_VERSION=v1.1.0
```

----

Before adding a new CNI, make sure to remove any old CNI:

```
rm -f /etc/cni/net.d/*.conf*
```

When starting the control plane node, make ensure that podCIDR is set:

```
kubeadm init --pod-network-cidr=10.244.0.0/16
```

When the node initialization is complete, the following will be shown:

```
You should now deploy a Pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

After the control plane is started, install the manifest:

```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.19.2/Documentation/kube-flannel.yml
```

```
docker.io/flannelcni/flannel-cni-plugin:v1.1.0
docker.io/rancher/mirrored-flannelcni-flannel-cni-plugin:v1.1.0  # mirror
docker.io/flannelcni/flannel:v0.19.2
docker.io/rancher/mirrored-flannelcni-flannel:v0.19.2  # mirror
```

Make sure that the CoreDNS pods are _running_ correctly:

```
kubectl get pods --all-namespaces
```


----

## Kubernetes Dashboard

### Installing dashboard and metrics-server

```
DASHBOARD_VERSION=v2.6.1
METRICS_SCRAPER_VERSION=v1.0.8
METRICS_SERVER_VERSION=v0.6.1
```

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.1/components.yaml
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```

```
docker.io/kubernetesui/dashboard:v2.6.1
docker.io/kubernetesui/metrics-scraper:v1.0.8
k8s.gcr.io/metrics-server/metrics-server:v0.6.1
registry.k8s.io/metrics-server/metrics-server:v0.6.1  # mirror
```

### Starting proxy and browser

```console
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

<http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login>

### Admin user and token

The main issue is creating the user and accessing the token, but it is also described in full detail in the docs...

<https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md>

```console
$ kubectl apply -f dashboard-adminuser.yaml
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
$ kubectl -n kubernetes-dashboard create token admin-user
...
```

### Self-signed certificate

You will still get the "Kubelet certificate needs to be signed by cluster Certificate Authority" from metrics-server.

<https://github.com/kubernetes-sigs/metrics-server#configuration>

```console
$ KUBE_EDITOR="sed -i '/args:/ a\ \ \ \ \ \ \ \ - --kubelet-insecure-tls'" kubectl edit deployment -n kube-system metrics-server
deployment.apps/metrics-server edited
```
