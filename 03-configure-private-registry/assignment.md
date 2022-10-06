---
slug: configure-private-registry
id: my3y43ajqkna
type: challenge
title: Configure Private Registry
teaser: Configure Private Registry
notes:
- type: text
  contents: |-
    This track uses a single node Kubernetes cluster on a sandbox virtual machine.

    ## Objectives

    In this challenge, this is what you'll learn:
    - Verify Nexus Registry Setup
    - Make the private registry available to Kubernetes
    - Run tests to ensure you are able to push and pull from the registry

    ## Duration

    ~30 minutes
tabs:
- title: Terminal 1
  type: terminal
  hostname: kubernetes-vm
- title: Gitea
  type: website
  path: /
  url: http://kubernetes-vm.${_SANDBOX_ID}.instruqt.io:30950
  new_window: true
- title: Argo CD
  type: website
  path: /
  url: http://kubernetes-vm.${_SANDBOX_ID}.instruqt.io:30080
  new_window: true
- title: Drone
  type: website
  path: /
  url: http://kubernetes-vm.${_SANDBOX_ID}.instruqt.io:30980
  new_window: true
- title: Terminal 2
  type: terminal
  hostname: kubernetes-vm
- title: Code
  type: code
  hostname: kubernetes-vm
  path: /root/repos/dag-stack
difficulty: basic
timelimit: 28800
---

🚀 Introduction
===============

As part of the challenges we will push the  images to container registries. To make it easy for push and pull, we wil use Sonatype Nexus Repository Manager(NXRM) that we deployed as part of the Argo CD apps.

🫙 Configure Private Registry
=============================

The NXRM Container registry is visible to the Kubernetes cluster. The container runtime has no clue about, to make the container runtime use it we need to configure that as  [k3s private registry](https://rancher.com/docs/k3s/latest/en/installation/private-registry/).

```shell
envsubst < "$DAG_HOME/config/etc/rancher/k3s/registries.yaml" | tee /etc/rancher/k3s/registries.yaml
```

Restart the k3s server to ensure it picks up the registry configuration,

```shell
systemctl restart k3s
```

Wait for kubernetes to be back up and running,

```shell
kubectl get pods -n kube-system
```

The output of the command should be like

&lt;pre&gt;
NAME                                      READY   STATUS      RESTARTS   AGE
local-path-provisioner-7b7dc8d6f5-p8jdb   1/1     Running     0          75m
coredns-b96499967-8vm62                   1/1     Running     0          75m
helm-install-traefik-crd-pk2bw            0/1     Completed   0          75m
helm-install-traefik-4fqwf                0/1     Completed   1          75m
svclb-traefik-89547357-7xclt              2/2     Running     0          75m
metrics-server-668d979685-865cp           1/1     Running     0          75m
traefik-7cd4fcff68-zmcn9                  1/1     Running     0          75m
&lt;/pre&gt;

Verify Registry Setup
=====================

Login into our private registry,

```shell
echo -n "admin123" | docker login -u admin  --password-stdin localhost:31081
```

Let us try to push an image to the private registry using docker,

```shell
docker pull gcr.io/google-samples/hello-app:1.0
docker tag gcr.io/google-samples/hello-app:1.0 "localhost:31081/hello-app:1.0"
docker push "localhost:31081/hello-app:1.0"
```

Let us use the image that we pushed earlier as part of Kubernetes deployment,

```shell
kubectl create deployment hello-server --image="nexus.infra.svc.cluster.local/hello-app:1.0"
kubectl rollout status deployment.apps/hello-server --timeout=30s
kubectl delete deployment.apps/hello-server
```

🏁 Finish
=========

To complete this challenge, press **Check**.
