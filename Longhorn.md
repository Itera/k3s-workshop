# Longhorn

This provides distributed storage.

It uses an area on each node by default - and you can add other disk volumes if needed.

You must have at least 3 instances for a volume to be regarded as "Healthy".

The easiest way to install this is via helm.

## Repository

Add the helm repository:

```shell
helm repo add longhorn https://charts.longhorn.io
helm repo update
```

## Install

Install:

```shell
helm install longhorn -n longhorn --create-namespace --version 1.6.1 longhorn/longhorn
```

## Ingress

Now we need access to the dashboard - install an ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-ingress
  namespace: longhorn
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web
spec:
  rules:
    - host: longhorn.k3s.workshop.itera.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: longhorn-frontend
                port:
                  number: 80
```

## Using the storage

### Deployment

Add a PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: foo-data
  namespace: foo
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: longhorn
  resources:
    requests:
      storage: 10Gi
```

And use it in the deployment image spec:

```yaml
volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: foo-data
```

Then mount it as a volumeMount:

```yaml
volumeMounts:
  - mountPath: /data
    name: data-volume
```

### StatefulSet

Add a volumeClaimTemplate and use it as a volumeMount in the container template spec section - for example:

```yaml
apiVersion: apps/v1
kind: StatefulSet
...
spec:
  ...
  template:
    spec:
      containers:
        - name: some container
          image: "some image"
          ...
          volumeMounts:
            - mountPath: /data
              name: data-volume
  volumeClaimTemplates:
    - metadata:
        name: data-volume
      spec:
        accessModes:
          - ReadWriteMany
        storageClass: longhorn
        resources:
          requests:
            storage: 5Gi
```
