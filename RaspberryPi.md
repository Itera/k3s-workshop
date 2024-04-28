# Raspberry Pi

You can run this just fine on one or more raspberry pis.

The Pi 5 works really well :)

## Architecture

The pi architecture is arm.

I would suggest going with the arm64 OS and installing it on an SSD (USB SSD is fine).

From [the docs](https://docs.k3s.io/installation/requirements?os=pi) - If deploying K3s with embedded etcd on a Raspberry Pi, it is recommended that you use an external SSD. etcd is write intensive, and SD cards cannot handle the IO load.

## Setup

You must enable cgroups.

From [the docs](https://docs.k3s.io/installation/requirements?os=pi):

Append `cgroup_memory=1 cgroup_enable=memory` to `/boot/cmdline.txt` after building the OS image before booting it.

You may also need to install vxlan support.

`apt install linux-modules-extra-raspi`

If its not found - check does `modprobe vxlan` return any errors - if not - you already have it.

## Multiplatform

Intel images from dockerhub will not run - it requires arm.

More and more images are being built multiplatform.

Here's a snippet of a github action that builds multiplatform:

```yaml
- name: Set up QEMU
  uses: docker/setup-qemu-action@v3

- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

- name: Login to DockerHub Registry
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    context: .
    platforms: linux/amd64,linux/arm64
    push: true
  tags: "${{ secrets.DOCKERHUB_USERNAME }}/${{ github.event.repository.name }}:${{ github.sha }},${{ secrets.DOCKERHUB_USERNAME }}/${{ github.event.repository.name }}:latest"
```

## Affinity

K3S sets a bunch of labels on each node by default.

Amongst them are some about the OS - e.g. on a Pi 5 it shows:

- kubernetes.io/arch=arm64
- kubernetes.io/os=linux

So - if you have a mixed cluster - some arm64 and some amd64 you can set node affinity:

For example - to force a deployment to an intel node:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: arch
              operator: In
              values:
                - amd64
```

This [blog post](https://blog.kubecost.com/blog/kubernetes-node-affinity/) was easy to understand.
