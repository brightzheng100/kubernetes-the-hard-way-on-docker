# Preparing All Kubernetes Nodes

All nodes are required to do some tuning and set up some requried tools.

It's recommended to log into all Kubernetes nodes (expect load balancers as they're not) and enable `iTerm2`'s `Toggle Broadcasting Input` feature, if you're using `iTerm2` like me, so that we can execute commands in one console and they will automatically broadcast to all others parallelly -- all the commands to be executed are the same in this lab.

But it's okay to execute the commands into following nodes one by one, if you don't mind the repetative work.

- k8s-master0
- k8s-master1
- k8s-master2
- k8s-worker0
- k8s-worker1

To log into containers, you may try:
- The native `Docker` way: `docker exec -it <node_name> bash`, e.g. `docker exec -it k8s-lb0 bash`


**All below steps will be executed within the node containers.**


## Tuning sysctl in all nodes

```sh
# Letting iptables see bridged traffic
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# Turning on source address verification
cat <<EOF | tee /etc/sysctl.d/10-network-security.conf
# Turn on Source Address Verification in all interfaces to
# prevent some spoofing attacks.
net.ipv4.conf.default.rp_filter=1
net.ipv4.conf.all.rp_filter=1
EOF

# Apply
sysctl --system
```

## Installing container runtime

We're going to use `contaierd` & `runc` here. Other container runtimes may work as well.

> Note: the `kind`-based image already has `containerd` and `runc` installed, but I still document/repeat the process here as eventually I will rebuild the image with only the bare minimum components to start with.

```sh
# Define desired components version
containerd_version=1.3.4
runc_version=1.0.0-rc90

# Install containerd
curl -sSLO https://github.com/containerd/containerd/releases/download/v${containerd_version}/containerd-${containerd_version}.linux-amd64.tar.gz
tar -C /usr/local -xvf containerd-${containerd_version}.linux-amd64.tar.gz
chmod +x /usr/local/bin/*
rm -f /usr/local/bin/containerd-stress /usr/local/bin/containerd-shim-runc-v1 containerd-1.3.4.linux-amd64.tar.gz

# Install runc
curl -sSLO https://github.com/opencontainers/runc/releases/download/v1.0.0-rc90/runc.amd64
chmod +x runc.amd64
mv runc.amd64 /usr/local/bin/runc
```

## Configuring container runtime

```sh
# setup systemd service for contaierd
cat > /etc/systemd/system/containerd.service <<EOF
# derived containerd systemd service file from the official:
# https://github.com/containerd/containerd/blob/master/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target
# disable rate limiting
StartLimitIntervalSec=0

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd
Restart=always
RestartSec=1

Delegate=yes
KillMode=process
Restart=always
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=1048576
# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity

[Install]
WantedBy=multi-user.target
EOF

# setup containerd config
mkdir -p /etc/containerd
cat > /etc/containerd/config.toml <<EOF
# ref: https://github.com/containerd/cri/blob/master/docs/config.md
version = 2

[plugins."io.containerd.grpc.v1.cri".containerd]
  default_runtime_name = "runc"
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  runtime_type = "io.containerd.runc.v2"

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.test-handler]
  runtime_type = "io.containerd.runc.v2"

[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "k8s.gcr.io/pause:3.2"
EOF

systemctl daemon-reload
systemctl start containerd
systemctl enable containerd
```

## Installing Kubernetes Components

```sh
apt-get update && apt-get install -y apt-transport-https gnupg
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF | tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

kubelet --version                   # Kubernetes v1.18.5, as of writing
kubeadm version -o short            # v1.18.5, as of writing
kubectl version --client --short    # Client Version: v1.18.5, as of writing

# Configure cgroup driver used by kubelet on control-plane node
mkdir -p /var/lib/kubelet
cat > /var/lib/kubelet/config.yaml <<EOF
# Ref: https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/kubelet/config/v1beta1/types.go
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd  # cgroupfs | systemd
failSwapOn: false
EOF

systemctl daemon-reload
systemctl restart kubelet
```

If you're in `iTerm2`'s broadcast mode, quit this mode now as we're done executing commands in all nodes.

Next: [04-bootstrap-control-plane](04-bootstrap-control-plane.md)
