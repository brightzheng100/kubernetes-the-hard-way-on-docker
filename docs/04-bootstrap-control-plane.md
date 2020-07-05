# Bootstrapping Control Plane

We can bootstrap Kubernetes in any of the control plane nodes.

Let's pick one control plane node, say `k8s-master0`, to bootstrap.

## Pick one control plane to bootstrap

> Note: In `k8s-master0`

```sh
# Define a kubeadm config file
mkdir -p /etc/kubernetes/kubeadm
cat > /etc/kubernetes/kubeadm/kubeadm-config.yaml <<EOF
# Ref: https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "k8s-lb0:6443"  # pointing to lb
networking:
  podSubnet: "10.32.0.0/12"           # the CIDR the CNI may prefer, here is weavenet
apiServer:
  certSANs:                           # extra SANs as we eventually will access the cluster from laptop
  - "localhost"
  - "127.0.0.1"
---
# Ref: https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/kubelet/config/v1beta1/types.go
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: cgroupfs  # cgroupfs | systemd
failSwapOn: false
EOF

# Bootstrap kubernetes
kubeadm init \
    --config=/etc/kubernetes/kubeadm/kubeadm-config.yaml \
    --ignore-preflight-errors=all \
    --upload-certs \
    --v=6
```

> SAMPLE OUTPUT:

```
...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join k8s-lb0:6443 --token r4ctec.u9nts6d7uf1nkbir \
    --discovery-token-ca-cert-hash sha256:1d070523f5f342240913c761f1519c78e6f69b8afcf70c4f10cecb5980464e34 \
    --control-plane --certificate-key 945a8f58211b8bc02f0f1feb6c27b163074611c32681064bc92d3245fe11234b

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-lb0:6443 --token r4ctec.u9nts6d7uf1nkbir \
    --discovery-token-ca-cert-hash sha256:1d070523f5f342240913c761f1519c78e6f69b8afcf70c4f10cecb5980464e34
```

Next: [05-join-nodes](05-join-nodes.md)
