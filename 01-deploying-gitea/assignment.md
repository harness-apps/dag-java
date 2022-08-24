---
slug: deploying-gitea
id: pyarbqplqky5
type: challenge
title: Deploy Gitea
teaser: Everything starts with git in GitOps world
notes:
- type: text
  contents: |-
    This track uses a single node Kubernetes cluster on a sandbox virtual machine.
    Please wait while we boot the VM for you and start Kubernetes.

    ## Objectives

    In this track, this is what you'll learn:
    - Deploy Gitea on Kubernetes
    - Expose the deployment with a NodePort service
    - View the exposed `gitea-http` service through a tab in Instruqt
    - Explore the Gitea dashboard
tabs:
- title: Terminal 1
  type: terminal
  hostname: kubernetes-vm
- title: Gitea
  type: website
  path: /
  url: http://kubernetes-vm.${_SANDBOX_ID}.instruqt.io:30950
  new_window: true
- title: Terminal 2
  type: terminal
  hostname: kubernetes-vm
difficulty: basic
timelimit: 600
---

👋 Introduction
===============

Ensure all required environment variables are set,

```shell
direnv allow .
```

Add Gitea helm charts,

```shell
helm repo add gitea-charts https://dl.gitea.io/charts/
helm repo update
```

Use the helm to deploy Gitea:

```shell
envsubst < "$DAG_HOME/helm_vars/gitea/values.yaml" | helm upgrade \
  --install gitea gitea-charts/gitea \
  --values - \
  --wait
```

👤 Add Gitea Users
==================

Gitea is configured with a default admin,

Username: ```demo```
Password: ```demo@123```

But for all the challenges we will be using the user `user-01`. The user by default will be configured with the following repositories,

- <https://github.com/harness-apps/dag-stack>
- <https://github.com/kameshsampath/drone-quickstart>
- <https://github.com/harness-apps/quarkus-springboot-demo>
- <https://github.com/harness-apps/quarkus-springboot-demo-gitops>

Run the following command to create `user-01` and configure the user with the repositories listed above.

```shell
kustomize build "$DAG_HOME/k8s/gitea-config" \
  | envsubst | kubectl apply -f -
```

Wait for the job to complete before proceeding further,

```shell
kubectl wait --for=condition=complete --timeout=120s -n drone job/workshop-setup
```

Now you can login to Gitea using Gitea tab using the user `user-01` and password `user-01@123`,

![Gitea Dashboard](../assets/gitea-user-dashboard.png)

Rename the existing remote to be `upstream`

```shell
git remote rename origin upstream
```

Add new remote to called `origin` to with url set to `${GITEA_URL}/user-01/dag-stack.git`

```shell
git remote add origin "${GITEA_DAG_REPO}"
```

Listing the git remotes,

```shell
git remote -v
```

```text
origin  ${GITEA_URL}/user-01/dag-stack.git (fetch)
origin  ${GITEA_URL}/user-01/dag-stack.git (push)
upstream        https://github.com/harness-apps/dag-stack.git (fetch)
upstream        https://github.com/harness-apps/dag-stack.git (push)
```

Push the current "${DAG_TARGET_VERSION}" branch to `origin`

```shell
git push origin "${DAG_TARGET_VERSION}"
```

🏁 Finish
=========

To complete this challenge, press **Check**.
