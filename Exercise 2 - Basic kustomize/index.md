# Exercise 2 - Basic kustomize

Start by deleting exercise 1

```shell
kubectl delete namespace workshop
```

## Exercise

Here the descriptor files are the same - but any labels, matchLabels and selectors are removed. This is a self-consistent set where we want the same labels across the board so we'll get kustomize to apply them for us.

You can install the kustomize binaries - but many kubectl apps have them available as `kubectl kustomize`

### Inspect what will be deployed

```shell
kubectl kustomize .
```

### Deploy

```shell
kubectl apply -k .
```
