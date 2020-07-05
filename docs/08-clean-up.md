# Cleaning Up

To clean up the env:

```sh
# delete the containers
for node in "k8s-lb0" "k8s-master0" "k8s-master1" "k8s-master2" "k8s-worker0" "k8s-worker1"; do
    docker rm -f ${node}
done

# delete the custom network
docker network rm lab
```

Congrats! You've done the Kubernetes the hard way, with `Kubeadm`, on Docker, within exactly your laptop!

Enjoy!
