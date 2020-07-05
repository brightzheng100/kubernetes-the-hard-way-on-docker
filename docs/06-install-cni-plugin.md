# Installing CNI Plugin

There are many Container Networking Interface (CNI) compliant plugins within the Kubernetes community.

Any of the compliant CNI plugins should work for basic use cases and it's your job to figure out what is right for you.

We can log into any of the control plane nodes to perform this task. For example:

```sh
docker exec -it k8s-master0 bash
```

Now we're in `k8s-master0`, do this:

```sh
# In k8s-master0
export KUBECONFIG=/etc/kubernetes/admin.conf

# If you prefer weave net
# See potential issues if you `kubeadm reset -f` then `kubeadm init` many rounds
# https://github.com/weaveworks/weave/issues/3634#issuecomment-652837244
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# Or cilium, choice is yours
kubectl create -f https://raw.githubusercontent.com/cilium/cilium/v1.8/install/kubernetes/quick-install.yaml
```

> Note: please install only **ONE** CNI for this case to avoid potential issues.

Wait for a while, the nodes should be in `Ready` state and the `pods` should be in `Running` state:

```sh
# Check the nodes
kubectl get nodes
```

> OUTPUT:

```
NAME          STATUS   ROLES    AGE     VERSION
k8s-master0   Ready    master   12m     v1.18.5
k8s-master1   Ready    master   8m24s   v1.18.5
k8s-master2   Ready    master   7m8s    v1.18.5
k8s-worker0   Ready    <none>   4m39s   v1.18.5
k8s-worker1   Ready    <none>   4m39s   v1.18.5
```

```sh
# Check the pods
kubectl get pod -n kube-system
```

> OUTPUT:

```
NAME                                  READY   STATUS    RESTARTS   AGE
coredns-66bff467f8-2hw7h              1/1     Running   0          12m
coredns-66bff467f8-hjdv5              1/1     Running   0          12m
etcd-k8s-master0                      1/1     Running   0          12m
etcd-k8s-master1                      1/1     Running   0          7m22s
etcd-k8s-master2                      1/1     Running   0          5m45s
kube-apiserver-k8s-master0            1/1     Running   0          12m
kube-apiserver-k8s-master1            1/1     Running   1          7m13s
kube-apiserver-k8s-master2            1/1     Running   3          5m55s
kube-controller-manager-k8s-master0   1/1     Running   1          12m
kube-controller-manager-k8s-master1   1/1     Running   0          7m44s
kube-controller-manager-k8s-master2   1/1     Running   0          6m18s
kube-proxy-5t6s5                      1/1     Running   0          7m23s
kube-proxy-6m4b5                      1/1     Running   0          4m54s
kube-proxy-786sh                      1/1     Running   0          4m54s
kube-proxy-lgrfj                      1/1     Running   0          12m
kube-proxy-vzt7d                      1/1     Running   0          8m39s
kube-scheduler-k8s-master0            1/1     Running   1          12m
kube-scheduler-k8s-master1            1/1     Running   0          7m31s
kube-scheduler-k8s-master2            1/1     Running   0          6m26s
weave-net-2sp7b                       2/2     Running   1          92s
weave-net-5q9nx                       2/2     Running   1          92s
weave-net-mqvmz                       2/2     Running   1          92s
weave-net-tqf4z                       2/2     Running   0          92s
weave-net-w6rcn                       2/2     Running   0          92s
```

Next: [07-access-it-from-laptop](07-access-it-from-laptop.md)
