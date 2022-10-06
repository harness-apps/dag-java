---
slug: deploying-argocd
id: wowgthfl57ue
type: challenge
title: Deploy Argo CD
teaser: Argo CD is de facto platform to do GitOps in kube world
notes:
- type: text
  contents: |-
    This track uses a single node Kubernetes cluster on a sandbox virtual machine.

    ## Objectives

    In this challenge, this is what you'll learn:
    - Deploy Argo CD on Kubernetes
    - Expose the Argo CD deployment with a NodePort service
    - View the exposed `argocd-server` service in your browser
    - Create Drone Server using the Argo CD "App of Apps" method
    - Expose the Drone CI deployment with a NodePort service
    - View the exposed `drone-ci` service in your browser
    - Explore the Drone CI dashboard

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
timelimit: 600
---

👋 Introduction
===============

GitOps is a way of implementing Continuous Deployment for cloud native applications. It focuses on a developer-centric experience when operating infrastructure, by using tools developers are already familiar with, including Git and Continuous Deployment tools.

The core idea of GitOps is having a Git repository that always contains declarative descriptions of the infrastructure currently desired in the production environment and an automated process to make the production environment match the described state in the repository. If you want to deploy a new application or update an existing one, you only need to update the repository - the automated process handles everything else. It’s like having cruise control for managing your applications in production.

> GitOps: versioned CI/CD on top of declarative infrastructure. Stop scripting > and start shipping.
>
> — [Kelsey Hightower](https://twitter.com/kelseyhightower/status/953638870888849408)

As part of this workshop we will be using [Argo CD](https://argo-cd.readthedocs.io/en/stable/user-guide/projects/) as our GitOps platform.

🐙 Deploy Argo CD
=================

Add Argo CD helm charts

```shell
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

Use the helm to deploy Argo CD,

```shell
envsubst < $DAG_HOME/helm_vars/argocd/values.yaml |\
  helm upgrade --install argocd argo/argo-cd \
    --create-namespace \
    --namespace=argocd \
    --values - \
    --wait
```

You can access Argo CD using the Argo CD tab. The default admin username is `admin` and password has been set to `demo@123`.

🔥 Cluster Bootstrapping
========================

The cluster bootstrapping installs the core DAG stack applications ([App of Apps](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#app-of-apps)) and DAG stack has the following child applications,

- Argo CD Image Updater
- Drone Server
- Droner Runners
- Artifactory as Maven Repository Manager

Environment Variables
---------------------

Few environment variables that use to bootstrap as shown below, they are already set in your shell via `$DAG_HOME/.envrc`,

&lt;pre&gt;
export GITEA_DOMAIN="kubernetes-vm.${DAG_BASE_DOMAIN}"
export GITEA_URL="http://${GITEA_DOMAIN}:30950"
export DRONE_SERVER_HOST="kubernetes-vm.$DAG_BASE_DOMAIN:30980"
export DRONE_SERVER_URL="http://${DRONE_SERVER_HOST}"
export GITEA_USER=user-01
&lt;/pre&gt;

Create Argo Apps
----------------

Update the DAG App `$DAG_HOME/helm_vars/dag/values.yaml` with values matching to the environment.

As we already got the required environment variables set, lets get them updated on the helm values,

```shell
envsubst < "$DAG_HOME/helm_vars/dag/values.tpl.yaml" > "$DAG_HOME/helm_vars/dag/values.yaml"
```

Commit and push the code to `${GITEA_DAG_REPO}`, so that values will be used by the DAG apps Argo CD application that values can be used by Argo Apps in upcoming sections.

```shell
git add "$DAG_HOME/helm_vars/dag/values.yaml"
git commit -m "Deploy DAG App of Apps" "$DAG_HOME/helm_vars/dag/values.yaml"
git push origin instruqt
```

Create DAG App on ArgoCD,

```shell
envsubst < "$DAG_HOME/k8s/dag/app.yaml" | kubectl apply -f -
```

Login to Argo server via cli,

```shell
argocd login "${ARGOCD_SERVER_HOST}" --plaintext --insecure --username admin --password='demo@123'
```

Trigger app sync

```shell
argocd app sync dag-apps
```

> **IMPORTANT**: The sync operation will take few minutes. Please wait for the sync to be complete before proceeding further.

A successful Argo CD deployment of apps should look as shown below,

![ArgoCD Apps](../assets/drag_apps.png)

✅ Validate Drone Setup
=======================

What we have done until now,

- Setup Gitea
- Setup `dag-apps` Argo CD that in turn setup
  - Drone Server
  - Drone Docker Runner
  - Nexus Registry
  - Argo CD Image Updater

Add Drone Admin User
--------------------

From the account settings named `Example CLI Usage` from the Drone Account Settings page,

![Drone Account Settings](../assets/drone_account_settings.png)

Copy the values,

```shell
export DRONE_SERVER="drone server as listed in the drone account settings page"
export DRONE_TOKEN="token from the drone account settings page"
```

Using the **Code** tab edit the `dag-stack` project and create a file named `.envrc.local` with the values of `DRONE_SERVER` and `DRONE_TOKEN` copied and exported from the account setting page

Ensure to reload the updated environment variables,

```shell
direnv allow .
```

Run the drone command to verify we are able to use drone cli,

```shell
drone user ls
```

As the current user `user-01` is not an admin, the command above fails with `client error 403: {"message":"Forbidden"}` message. The current user need to be an admin user to be able to perform few administrative tasks as part of other challenges.

Edit the `.envrc.local` file and add the following environment variable,

```shell
export ENABLE_DRONE_ADMIN=true
```

Update the DAG App `$DAG_HOME/helm_vars/dag/values.yaml` with values matching to the environment,

```shell
envsubst < "$DAG_HOME/helm_vars/dag/values.tpl.yaml" > "$DAG_HOME/helm_vars/dag/values.yaml"
```

Commit and push the code,

```shell
git commit -m "Enable Admin" "$DAG_HOME/helm_vars/dag/values.yaml"
git push origin "${DAG_TARGET_VERSION}"
```

Go to the **Argo CD** tab and watch the "dag-apps", in few seconds you could see Argo CD rollout update to `drone` app.

> **NOTE**: You can also click on refresh to see the updates.

Running `drone user ls` command again now be successful and listing the only user `user-01`.

```shell
drone user ls
```

You can also verify of the `user-01` is an admin or not by running the command,

```shell
drone info --format "Admin: {{ .Admin }}"
```

The command should show an output like,

&lt;pre&gt;Admin: true&lt;/pre&gt;

🏁 Finish
=========

To complete this challenge, press **Check**.
