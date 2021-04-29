# Flux v1 helm-operator with charts from git source

This repository evaluates flux v1's [helm-operator](https://github.com/fluxcd/helm-operator) installing charts from git sources using git refs as the only way of specifying the version of the chart to be installed. 

With flux v2, [source-controller](https://github.com/fluxcd/source-controller) and [helm-controller](https://github.com/fluxcd/helm-controller) and some changes to how helm requires chart versions, this might behave differently. This repository is meant to be used to test migrations from flux v1 to v2 and so from helm-operator to helm-controller.

PR tracking the feature to allow similar behaviour with flux v2: https://github.com/fluxcd/source-controller/pull/308

## Structure

A single branch tracks the `HelmRelease`s for all stages. That branch could be synced by flux. Charts are managed in the same repository, referenced via git sources from the `HelmRelease`s. The environments are represented as different namespaces. One `helm-operator` instances manages the releases for both environments.

The dev `HelmRelease` will reference the branch HEAD to immediately rollout the latest changes. For prod, the `HelmRelease` points to an older commit (referenced via git tag). Changes to the chart will immediately be applied to dev but require a new git tag and an update of the `HelmRelease` for prod.

The chart's own version number remains unchanged. Helm will report that as 0.1.0.

## Setup

To verify this setup:

- create a deploy key for the git repo
- install the latest `helm-operator` (1.2.0) supplying the key
- create two namespaces and apply the corresponding `HelmRelease`s



### Helm operator

```
k create ns flux

helm repo add fluxcd https://charts.fluxcd.io

ssh-keygen -q -N "" -f ./identity

k create secret generic helm-operator-ssh \
    --from-file=./identity \
    --namespace flux

helm upgrade -i helm-operator fluxcd/helm-operator \
    --namespace flux \
    --version 1.2.0 \
    --set helm.versions=v3 \
    --set git.ssh.secretName=helm-operator-ssh
```

### Install HelmReleases

dev (aka latest and greatest helm chart)

```
k apply -f releases/dev/namespace.yaml
k apply -f releases/dev/simple-nginx.helmrelease.yaml
```

prod (aka manually selected commit via git tag)

```
k apply -f releases/prod/namespace.yaml
k apply -f releases/prod/simple-nginx.helmrelease.yaml
```

### Verify

The environment variable `fixedenvval` is managed in the chart only. The `envval` is overwritten by the `HelmRelease` that was applied.

```
helm -n helm-test-fake-dev get manifest simple-nginx | yq eval '.spec.template.spec.containers[0].env' -
helm -n helm-test-fake-prod get manifest simple-nginx | yq eval '.spec.template.spec.containers[0].env' -
```
