# Accessing Kubernetes from the Laptop / Host

There is already one kubeconfig file generated in all nodes, as `/etc/kubernetes/admin.conf`.

We can copy it back to our laptop, the host, so we can access it in an much convenient way.

Within our laptop, or the host:

```sh
# Copy it out, change the config name if you want
docker cp k8s-master0:/etc/kubernetes/admin.conf ~/.kube/config-lab

# We need to update the server as it mentions `server: https://k8s-lb0:6443`
# so we need to get the port mapping, e.g. 6443/tcp -> 0.0.0.0:32775
port="$( docker port k8s-lb0 6443 | cut -d":" -f2 )"
# In Mac, do this
sed -i '' "s|server: https://k8s-lb0:6443|server: https://localhost:${port}|g" ~/.kube/config-lab
# In Linux
sed -i "s|server: https://k8s-lb0:6443|server: https://localhost:${port}|g" ~/.kube/config-lab

export KUBECONFIG=~/.kube/config-lab
kubectl get nodes
kubectl get pods -n kube-system
```

You should see exactly the same result as what you did in any of the Kubernetes nodes.

Next: [08-clean-up](08-clean-up.md)
