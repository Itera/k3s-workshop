# Exercise 3 - Secrets and SealedSecrets

Make sure the workspace from Exercise 2 is currently running.

## Creating secrets

Creating with kubectl

```shell
kubectl -n <namespace> create secret generic <secret-name> \
--from-file=<file> \
--from-literal=name=value
```

Or - via descriptor:

```shell
kubectl apply -f secret.yaml
```

Note that the secret itself is only base64 encoded. Not great for public repositories - but SealedSecrets will help with that.

Let's remove it and then seal it.

Removal:

```shell
kubectl delete -f secret.yaml
```

Types of secrets - see https://kubernetes.io/docs/concepts/configuration/secret/#secret-types - here we've just used an Opaque secret.

## Sealed Secrets

This requires two things - something in the kube-system cluster and a command line application.

Full info: https://github.com/bitnami-labs/sealed-secrets/blob/main/README.md#installation

### Controller

But - we can start by simply running:

```shell
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.26.2/controller.yaml
```

That should configure up a lot of stuff in kube-system namespace.

### Command line app

You will need the kubeseal app - see the readme linked above.

### Get your public key

SealedSecrets is a private/public key system.

Any secret signed by the public key can be safely stored in github or similar - and then on deploy will be picked up by the controller - and decrypted using the local private key - to a normal kubernetes secret.

To do that - we need the public key

```shell
kubeseal --fetch-cert \
--controller-name=sealed-secrets-controller- \
--controller-namespace=kube-system \
> pub-sealed-secrets.pem
```

### Sealing your secret

```shell
kubeseal --format=yaml --cert=pub-sealed-secrets.pem < secret.yaml > secret-sealed.yaml
```

Inspect the new secret-sealed.yaml and then apply it.

```shell
kubectl apply -f secret-sealed.yaml
```

Check that both the sealed secret is deployed, and that it is complete - and that the decrypted secret is also deployed and available.

**Note**

If you are going to use a sealed secret in a different namespace than it was originally signed for (think you deploy various versions of an app or similar) you will need to add `--scope cluster-wide` to the call to kubeseal - this adds an annotation to the sealed secret file.

It is not enough just to change the namespace in the sealed-secret file - it is also part of what was enrypted so you must tell the controller that its OK to use a different one by saying it can be used elsewhere in the cluster.
