# K3S workshop

## VM setup

See files in [VM setup](VM%20setup/) folder.

We will need minimum one linux node - if you want to test out multiple agents - you will need several.

## Installation

See [Installation](Installation.md) for instructions on installing K3S

K3s install includes kubectl - for testing from your local machine - you'll need:

- Kubectl
- Helm
- K9S (not needed but I like it as a tool)

## Exercises

- [Exercise 1 - Basic kubectl](Exercise%201%20-%20Basic%20kubectl/) - test that your server is running with basic kubectl commands
- [Exercise 2 - Basic kustomize](Exercise%202%20-%20Basic%20kustomize/) - repeat exercise 1 but use kustomize
- [Exercise 3 - Secrets and SealedSecrets](Exercise%203%20-%20Secrets%20and%20SealedSecrets/) - installing secrets and sealing them
- [Exercise 4 - Metrics](Exercise%204%20-%20Metrics/) - setup of prometheus and grafana
- [Exercise 5 - Traefik Dashboard](Exercise%205%20-%20Traefik/) - setup of traefik dashboard

## Examples

Some things that are hard to run in a workshop setup but can work if you have a cluster.

- [Letsencrypt - setting up certificate support](LetsEncrypt.md)
- [Longhorn - shared disk](Longhorn.md)
- [Flux - devops deployment](Flux.md)
