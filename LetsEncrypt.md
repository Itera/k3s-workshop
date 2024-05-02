# LetsEncrypt SSL

The easiest way to install this is via helm.

First stage is to install cert-manager.

## Cert-Manager

### Repository

Add the helm repository:

```shell
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

### Install

Set up values - for this example - grabbed from [k3s.rocks](https://raw.githubusercontent.com/askblaker/k3s.rocks/main/manifests/prometheus-values.yaml)

Then install the chart - using the values file:

```shell
helm install cert-manager -n cert-manager --create-namespace --version 1.14.4 --set installCRDs=true jetstack/cert-manager
```

## Issuers

We will install an issue that can use LetsEncrypt in http mode.

Modify the email and then apply this with kubectl:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: <an email>
    privateKeySecretRef:
      name: letsencrypt-private-key
    solvers:
      - http01:
          ingress:
            class: traefik
```

### Notes

About namespaces - https://cert-manager.io/docs/configuration/#cluster-resource-namespace

There is a [staging endpoint available](https://letsencrypt.org/docs/staging-environment/) - it uses the server url: https://acme-staging-v02.api.letsencrypt.org/directory

The acme issuer also supports dns01 as well as http01 - for letsencrypt DNS challenge - https://letsencrypt.org/docs/challenge-types/

## Using it

Recommended is installing an http to https redirect:

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: redirect-https
  namespace: some_namespace
spec:
  redirectScheme:
    scheme: https
    permanent: true
```

Then in the ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app_name
  namespace: some_namespeace
  annotations:
    spec.ingressClassName: traefik
    cert-manager.io/cluster-issuer: letsencrypt
    traefik.ingress.kubernetes.io/router.middlewares: some_namespace-redirect-https@kubernetescrd
spec:
  rules:
    - host: host.domain.tld
      http:
        paths:
        ... // Point to a service and port
  tls:
    - secretName: app_name-tls
      hosts:
        - host.domain.tld
```

Cert-Manager will then go and get a new certificate from letsencrypt as and when needed.
