# Installation of k3s

If you want repeatable, scriptable, commit to git type of cluster setup - take a look at [k3d](https://k3d.io) - but for the workshop we'll use the quick start script from get.k3s.io

On the main node - as root

```shell
curl -sfL https://get.k3s.io | sh -
```

On any agent nodes - first copy the file `/var/lib/rancher/k3s/server/node-token` to each agent from the main node:

```shell
scp /var/lib/rancher/k3s/server/node-token agent-hostname:
```

Then - as root on each agent:

```shell
curl -sfL https://get.k3s.io | K3S_URL=https://main-hostname:6443 K3S_TOKEN=TOKEN_FROM_FILE sh -
```

Agent nodes can now just be left running in the background.

## Did it work?

```shell
kubectl get nodes
```

## Kube config

kubectl will work out of the box - but some other things (k9s - see below) will look in .kube/config or the env KUBECONFIG location.

In general - copy `/etc/rancher/k3s/k3s.yaml` from the main node to a local file then set the environment variable:

```shell
export KUBECONFIG=path/to/k3s.yaml
```

Lastly - edit the file and change the server address.

It even works from your main laptop outside the VM - as long as you can connect to 192.168.64.2:6443 and have kubectl installed locally.

## K9s

Install k9s - either from https://k9scli.io or from derailed/k9s releases or via any package manager you use that contains it.

## Caveats

### Issues due to running in a VM setup

- Login as root is likely disabled - most commands will need to be run in a terminal where you have switched to the root user - `sudo su -`
- SSH in as root is likely disabled - so to copy files in - copy to your non-root user - root can see the files anyway
- Hostnames will only work if you setup your /etc/hosts files - use IP addresses if not

Remember that all of this is caused by running in a temporary setup - and is not due to k3s itself.
