---
title: Setup Kubernetes with kubeadm in CentOS VMs
date: 2018-11-03 19:09:30
categories: Cloud
tags:
  - Kubernetes
---

Here is a guide on how to setup Kubernetes on two CentOS 7 VMs (master and node).

1. Stop firewall/selinux (both)
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

2. Ensure swap is off. Add comment in fstab (both)
```
#/dev/mapper/centos-swap swap                    swap    defaults        0 0
```
Execute `swapoff -a`.

<!-- more -->

3. Enable IP forwarding or iptables. (both)
```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

4. Install Docker (both). Reference: https://docs.docker.com/install/linux/docker-ce/centos/
```
# sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# sudo yum install -y docker-ce
# sudo systemctl start docker
# sudo systemctl enable docker
```

5. Installing kubeadm, kubelet and kubectl (both). Reference: https://kubernetes.io/docs/setup/independent/install-kubeadm/
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable kubelet && systemctl start kubelet
```

6. Initializing your master (master)
```
kubeadm init

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

export KUBECONFIG=/etc/kubernetes/admin.conf
```

7. Installing a pod network add-on (master)  (Wave Net)
```
sysctl net.bridge.bridge-nf-call-iptables=1
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
kubectl get pods --all-namespaces
```

8. Joining your nodes(node)
```
kubeadm join <master-ip>:6443 --token zt5yjr.zutab0w4uczftlm8 --discovery-token-ca-cert-hash sha256:a34dc33567e0e169cdc0366303fb409d030dfd01602973ae482e9ef4969f2962
```

9. Dashboard (master)
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

10. storage (master). Rook
```
$ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/operator.yaml

$ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/cluster.yaml
$ kubectl get pods -n rook-ceph-system

$ kubectl get pods -n rook-ceph
```

11.Deploy first app
```
# vi nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
# kubectl create -f nginx-deployment.yaml
# kubectl get pods
```

12.Expose a service
```
# kubectl expose deployment nginx-deployment --type=LoadBalancer
# kubectl get services
```
