# Setup

## Helm operator

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

## Releases

```
k apply -f releases/dev/namespace.yaml
k apply -f releases/dev/simple-nginx.helmrelease.yaml

helm get manifest simple-nginx
```
