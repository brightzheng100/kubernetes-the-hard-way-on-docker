# Spinning Up Containers as Kubernetes Nodes

Here we're going to spin up a couple of contaiers, as Kubernetes "nodes":

- 1 x Load Balancer node
- 3 x Control Plane nodes
- 2 x Worker nodes

A dedicated Docker network will be used too for hostname based communication:

```sh
# Let's create a dedicated Docker network
docker network create lab
```

## Use any of the approaches

### By bash scripts

```sh
# Spin up the LB
docker run \
    --name "k8s-lb0" \
    --hostname "lb0" \
    --network lab \
    --detach \
    --restart=on-failure:1 \
    --tty \
    --publish=6443/TCP \
    quay.io/brightzheng100/k8s-haproxy:2.1.7-alpine

# Spin up the Kubernetes nodes
for node in "master0" "master1" "master2" "worker0" "worker1"; do
    docker run \
        --name "k8s-${node}" \
        --hostname "${node}" \
        --network lab \
        --privileged \
        --security-opt seccomp=unconfined \
        --security-opt apparmor=unconfined \
        --detach \
        --restart=on-failure:1 \
        --tty \
        --tmpfs /tmp \
        --tmpfs /run \
        --tmpfs /run/lock \
        --volume /var \
        --volume /lib/modules:/lib/modules:ro \
        --volume /sys/fs/cgroup:/sys/fs/cgroup:ro \
        quay.io/brightzheng100/k8s-ready:ubuntu.20.04
done

# Review the containers we spun up
docker ps --format '-->  {{.Image}} -- {{.Names}}'
```

> OUTPUT: we should see all required "nodes" available

```
-->  quay.io/brightzheng100/k8s-haproxy:2.1.7-alpine -- k8s-lb0
-->  quay.io/brightzheng100/k8s-ready:ubuntu.20.04 -- k8s-worker1
-->  quay.io/brightzheng100/k8s-ready:ubuntu.20.04 -- k8s-worker0
-->  quay.io/brightzheng100/k8s-ready:ubuntu.20.04 -- k8s-master2
-->  quay.io/brightzheng100/k8s-ready:ubuntu.20.04 -- k8s-master1
-->  quay.io/brightzheng100/k8s-ready:ubuntu.20.04 -- k8s-master0
```

### Or by `footloose`

TODO

## Update the haproxy config

```sh
# Update the haproxy accordingly and reload the config
# Note: ubuntu image -> /etc/haproxy/haproxy.cfg; alpine image -> /usr/local/etc/haproxy/haproxy.cfg
docker exec -i k8s-lb0 sh <<'EOF'
cat > /usr/local/etc/haproxy/haproxy.cfg <<__EOF
frontend controlPlane
    bind 0.0.0.0:6443
    mode tcp
    default_backend kube-apiservers

backend kube-apiservers
    mode tcp
    server k8s-master0 k8s-master0:6443
    server k8s-master1 k8s-master1:6443
    server k8s-master2 k8s-master2:6443
__EOF

kill -s HUP 1

exit
EOF
```

Next: [03-prepare-all-nodes](03-prepare-all-nodes.md)
