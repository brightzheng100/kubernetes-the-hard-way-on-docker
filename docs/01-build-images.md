# Building Images

There are two Docker images involved:

- **haproxy**, which acts as the load balancer, with `sshd` installed;
- **k8s-ready**, which will be Kubernetes ready as the "nodes", with `systemd` as the init system and `containerd` installed

## haproxy

```sh
( cd images/haproxy && docker build -t quay.io/brightzheng100/k8s-haproxy:2.1.7-alpine . )
```

## k8s-ready

We're going to use `kind`'s [base image](https://github.com/kubernetes-sigs/kind/tree/master/images/base) with some small changes:

- Install extra components: `vim-tiny`
- Remove some pre-installed components like `containerd`, as we will install them during the process
- `mkdir /kind` to make the image happy as we don't use `kind create cluster` to automate the cluster creation process

```sh
( cd images/k8s-ready && docker build -t quay.io/brightzheng100/k8s-ready:ubuntu.20.04 . )
```

> Note: 
> 1. Please change the tagging accordingly if you want
> 2. I've already pushed these images to `quay.io/brightzheng100`, so you may use them directly without the need to build.


Next: [02-spinup-containers-as-nodes](02-spinup-containers-as-nodes.md)
