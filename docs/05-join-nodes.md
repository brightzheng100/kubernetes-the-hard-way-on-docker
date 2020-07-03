# Joining Nodes to Cluster

## Joining control plane nodes

Run following command in both `k8s-master1` and `k8s-master2`:

```sh
# This command is copied from the output of `kubeadm init`, with --ignore-preflight-errors=all
kubeadm join k8s-lb0:6443 --token r4ctec.u9nts6d7uf1nkbir \
    --discovery-token-ca-cert-hash sha256:1d070523f5f342240913c761f1519c78e6f69b8afcf70c4f10cecb5980464e34 \
    --control-plane --certificate-key 945a8f58211b8bc02f0f1feb6c27b163074611c32681064bc92d3245fe11234b \
    --ignore-preflight-errors=all
```

Let's check the nodes in any of the control plane nodes:

```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl get nodes
```

> OUTPUT:

```
NAME      STATUS     ROLES    AGE     VERSION
master0   NotReady   master   8m54s   v1.18.5
master1   NotReady   master   4m17s   v1.18.5
master2   NotReady   master   74s     v1.18.5
```

> Note: Why they're `NotReady`? It's because they're not yet installed with the CNI plugin! We'll be there soon.


## Joining worker nodes

The `kubeadm init` generates another command for joining worker nodes too.

Let's run following command in both `k8s-worker0` and `k8s-worker1` nodes:

```sh
# This command is copied from the output of `kubeadm init`, with --ignore-preflight-errors=all
kubeadm join k8s-lb0:6443 --token r4ctec.u9nts6d7uf1nkbir \
    --discovery-token-ca-cert-hash sha256:1d070523f5f342240913c761f1519c78e6f69b8afcf70c4f10cecb5980464e34 \
    --ignore-preflight-errors=all
```

If you check the nodes again, you would see more nodes:

```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl get nodes
```

> OUTPUT:

```
NAME      STATUS     ROLES    AGE     VERSION
master0   NotReady   master   14m     v1.18.5
master1   NotReady   master   9m55s   v1.18.5
master2   NotReady   master   6m52s   v1.18.5
worker0   NotReady   <none>   7s      v1.18.5
worker1   NotReady   <none>   2m      v1.18.5
```

Next: [06-install-cni-plugin](06-install-cni-plugin.md)
