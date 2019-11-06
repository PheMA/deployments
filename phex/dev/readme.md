# 🏃🏻‍ PhEx Dev Deployment

<a href="https://projectphema.org" alt="PhEx"><img src="../repo-badge.svg"/></a>

A development deployment of the PhEx application.

Flux was run on the cluster as follows:

```sh
helm upgrade -i flux \
--set helmOperator.create=true \
--set helmOperator.createCRD=false \
--set git.url=git@github.com:phema/deployments \
-—set git.path=phex/dev \
--namespace flux \
fluxcd/flux
```

There is a Helm chart in [`charts/phema-phex`](./charts/phema-phex/) that
descibes a basic deployment of the PhEx fat JAR using the [published Docker
image](https://bintray.com/beta/#/phema/docker/phema-phex?tab=overview).

[`cert-manager`](https://docs.cert-manager.io/) is used to generate SSL
certificates using [Let's Encrypt](https://letsencrypt.org/).

The [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/) is
 deployed, and there is an Ingress created to route to the PhEx service.

Basic auth is set up on the Ingress, so there needs to be a secret created as
follows:

```sh
$ htpasswd -bc auth USERNAME PASSWORD

$ kubectl create secret generic phex-basic-auth --from-file=auth -n phema
```