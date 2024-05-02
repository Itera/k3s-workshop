# Exercise 5 - Traefik

K3S comes with Traefik as ingress controller by default.

But the dashboard is not exposed - so either a portforward is needed or a service and ingress.

These can be applied with kubectl

## Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: traefik-dashboard
  namespace: kube-system
  labels:
    app.kubernetes.io/instance: traefik
    app.kubernetes.io/name: traefik-dashboard
spec:
  ports:
    - name: traefik
      port: 9000
      targetPort: traefik
      protocol: TCP
  selector:
    app.kubernetes.io/instance: traefik-kube-system
    app.kubernetes.io/name: traefik
```

## Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: traefik-ingress
  namespace: kube-system
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web
spec:
  rules:
    - host: traefik.domain.tld
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: traefik-dashboard
                port:
                  number: 9000
```

### Ingress with a static TLS certificate

For letsencrypt and cert-manager see [letsencrypt](../LetsEncrypt.md)

But here - a static certificate.

Create a secret with the key and certificate.

```shell
kubectl create secret generic tls --from-file=tls.key --from-file=tls.crt -n namespace --dry-run=client --output=yaml > secret.yaml
```

Then apply it (or seal it with kubeseal and apply the sealed secret):

```shell
kubeseal --format=yaml  --cert=pub-sealed-secrets.pem < secret.yaml > sealed-secret.yaml
```

With these in place - change the ingress - use this annotation instead of web

```yaml
traefik.ingress.kubernetes.io/router.entrypoints: websecure
```

And add this to the spec:

```yaml
tls:
  - secretName: tls
```

## Basic Auth

For some internal work - basic auth may be enough.

We need to add a Middleware.

It will need a secret that contains the basic auth configuration.

More info - https://doc.traefik.io/traefik/middlewares/http/basicauth/

But the basics (here it is assumed that namespace is set by the kubectl command):

### Create a users file

Create a user auth file with htpasswd - e.g.:

```shell
htpasswd -c userfile username
htpasswd userfile username2
htpasswd userfile username3
```

Base64 encode the file

The file [testusers](testusers) contains

- testuser1:testuser1
- testuser2:testuser2
- testuser3:testuser3

### Users file secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: basic-auth-secret
data:
  users: |2
    BASE64_ENCODED_FILE_HERE
```

### Add the middleware needed

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: basic-auth
spec:
  basicAuth:
    secret: basic-auth-secret
```

### Applying to ingress

Use an annotation.

The value will be `namespace-middlware_name@kubernetescrd`

```yaml
traefik.ingress.kubernetes.io/router.middlewares: namespace-middlware_name@kubernetescrd
```
