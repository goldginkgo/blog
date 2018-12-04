---
title: Setup Kuberntes using minikube on MacOS
date: 2018-11-01 20:01:06
tags:
  - Kubernetes
---
Here is guide on how to setup Kubernetes 1.12 using minikube 0.30 on MacOS.
<!-- more -->
### Setup Shadowsocks
Setup shadowsocks, change shadowsocks "HTTP proxy listen ip" to local ip. e.g 192.168.1.107. Use 'ipconfig getifaddr en0' command to check IP.

### Set proxy in terminal
Open terminal, set proxy.
```
export http_proxy=http://192.168.1.107:1087;export https_proxy=http://192.168.1.107:1087;export no_proxy=localhost,127.0.0.0/8,192.0.0.0/8;
```

### Install and start docker

### Setup minikube
Execute following command to setup minikube.
```
minikube start --vm-driver=hyperkit --kubernetes-version v1.12.1 --docker-env HTTP_PROXY=http://192.168.1.107:1087 --docker-env HTTPS_PROXY=http://192.168.1.107:1087 --docker-env NO_PROXY=localhost,127.0.0.0/8,192.0.0.0/8;
```

### Commands for debugging
Here are a list of commands that maybe useful for debugging the setup if error occurs.
```
minikube ssh
docker ps -a
docker logs <container>

kubectl get pods --all-namespaces
kubectl logs -n kube-system <pod>
```

### Open dashboard
Open Kubenetes dashboard using `minikube dashboard` command.

### Cleanup
If you want to delete minikube and Kubernetes, use following command.
```
minikube stop && minikube delete && rm -rf ~/.minikube ~/.kube
```
