# PhEx Dev Deployment

A deployment for the team to share during development.

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