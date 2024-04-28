# Exercise 4 - Metrics

The easiest way to install this is via helm.

## Repository

Add the helm repository:

```shell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

## Install

Set up values - for this example - grabbed from [k3s.rocks](https://raw.githubusercontent.com/askblaker/k3s.rocks/main/manifests/prometheus-values.yaml)

Then install the chart - using the values file:

```shell
helm install prometheus-stack -n metrics --create-namespace --version 58.2.1 -f values.yaml prometheus-community/kube-prometheus-stack
```

## Ingress

Now we need access - install the ingresses - this installs 3 - one for alert-manager, one for prometheus and one for grafana:

```shell
kubectl apply -f ingress.yaml
```

Remember to test these you'll need to be able to resolve the relevant hostnames (at least locally).
