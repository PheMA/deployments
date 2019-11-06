# 🚀 PhEMA Deployments

> Config for [GitOps](https://www.weave.works/technologies/gitops/) deployments

<p align="center">
    <a href="https://github.com"><img src="./img/github.svg" width="120px"></a>
    <a href="https://www.docker.com"><img src="./img/docker.svg" width="120px"></a>
    <a href="https://kubernetes.io"><img src="./img/kubernetes.svg" width="120px"></a>
    <a href="https://helm.sh"><img src="./img/helm.svg" width="120px"></a>
    <a href="https://docs.fluxcd.io"><img src="./img/flux.png" width="120px"/></a>
</p>

This repository contains deployment configs for PhEMA deployments. The intention
is for each deployment to have all the necessary config in its own directory.
For example, [`phex/dev`](./phex/dev) contains everything necessary to recreate
the PhEx development server deployment.

## Overview

The main idea is to use [GitOps](https://www.weave.works/technologies/gitops/)
to easily automate deployments. Our software libraries and applications are
automatically published to publicly accessibly servers when tags are pushed to
their respective repos, which is achieved via their Travis configs (e.g.
[here](https://github.com/PheMA/phex/blob/d1cf8a73d275ec43d574dcfcc46043d1cd53e75e/.travis.yml)).
We then describe the desired end state of our deployments using Kubernetes
resources and Helm charts in this repository. Finally, we run the
[Flux](https://docs.fluxcd.io/en/stable/) agent in a Kubernetes cluster
configured to watch a specific path in this repository. Flux will take care of
making sure the deployed config matched what is in this repo.

## Setup

There are few manual steps to get started. These docs will assume you have a
Linux VM already provisioned, with ports `80`, `443`, and `22` open.

### <img src="./img/docker.svg" width="15px"> Install Docker

The first step is to install Docker. The Community Edition is sufficient. There
are install instructions for many platforms
[here](https://docs.docker.com/v17.09/engine/installation/). The Ubuntu install
instructions are
[here](https://docs.docker.com/v17.09/engine/installation/linux/docker-ce/ubuntu/).

### <img src="./img/kubernetes.svg" width="15px"> Setup Cluster

The next step is to set up the Kubernetes cluster. First, install `kubeadm` by
following the instructions [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl).

Initialize the cluster:

```sh
$ sudo kubeadm init
```

Then, do what it says at the end of the output, which will be something like:

```sh
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

The output will direct you [here](https://kubernetes.io/docs/concepts/cluster-administration/addons/) and tell you to install a network manager, plugin.
To install the weave network manager, run:

```sh
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

### <img src="./img/helm.svg" width="15px"> Install Helm

Next, install Helm by following [these instructions](https://helm.sh/docs/using_helm/#install-helm). Then configure it:

```sh
$ kubectl -n kube-system create sa tiller

$ kubectl create clusterrolebinding tiller-cluster-rule \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:tiller
```

Then install Tiller, the in-cluster Helm agent into the cluster:

```sh
$ helm init --skip-refresh --upgrade --service-account tiller --history-max 10
```

### <img src="./img/flux.png" width="15px"> Install Flux

Install Flux using Helm:

```sh
$ helm repo add fluxcd https://charts.fluxcd.io
```

Install the Flux [custom resource definitions (CRDs)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions):

```sh
kubectl apply -f https://raw.githubusercontent.com/fluxcd/flux/helm-0.10.1/deploy-helm/flux-helm-release-crd.yaml
```

Finally, install Flux to watch a specific path in this repo:

```sh
helm upgrade -i flux \
	--set helmOperator.create=true \
	--set helmOperator.createCRD=false \
	--set git.url=git@github.com:phema/deployments \
	--set git.path="phex/dev" \
	--namespace flux \
	fluxcd/flux
```

:bulb: **Note the `phex/dev` value for the `git.path` parameter. Only resources
at that path in this repo will be synchronized to the cluster.**

### 🔐 Install `cert-manager`

If you intend to use `cert-manager` to generated certificates (recommended), then
also install the `cert-manager` CRDs:

```sh
kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml
```

You can now begin adding Kubernetes resources to the appropriate path in this
repo and Flux will deploy them to the cluster.

## Troubleshooting

It is highly recommend to install [stern](https://github.com/wercker/stern) for
Kubernetes logging.

If you need to completely reset the cluster, you can run:

```sh
$ sudo kubeadm reset -f

$ rm -rf $HOME/.kube
```