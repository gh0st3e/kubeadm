KUBERNETES 1.28

CONTAINERD 1.6.16

UBUNTU 22.04

# PRE-INSTALL STEPS: [FOR BOTH NODES]

`printf "\n192.168.15.93 k8s-control\n192.168.15.94 k8s-2\n\n" >> /etc/hosts`

`printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/containerd.conf`

`modprobe overlay`

`modprobe br_netfilter`

`printf "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\n" >> /etc/sysctl.d/99-kubernetes-cri.conf`

`sysctl --system`

`wget https://github.com/containerd/containerd/releases/download/v1.6.16/containerd-1.6.16-linux-amd64.tar.gz -P /tmp/`

`tar Cxzvf /usr/local /tmp/containerd-1.6.16-linux-amd64.tar.gz`

`wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -P /etc/systemd/system/`

`systemctl daemon-reload`

`systemctl enable --now containerd`

`wget https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64 -P /tmp/`

`install -m 755 /tmp/runc.amd64 /usr/local/sbin/runc`

`wget https://github.com/containernetworking/plugins/releases/download/v1.2.0/cni-plugins-linux-amd64-v1.2.0.tgz -P /tmp/`

`mkdir -p /opt/cni/bin`

`tar Cxzvf /opt/cni/bin /tmp/cni-plugins-linux-amd64-v1.2.0.tgz`

`mkdir -p /etc/containerd`

`containerd config default | tee /etc/containerd/config.toml`

`nano /etc/containerd/config.toml`   <<<<<<<<<<< manually edit and change `systemdCgroup` to true

`systemctl restart containerd`

`swapoff -a`  <<<<<<<< just disable it in /etc/fstab instead

`apt-get update`

`apt-get install -y apt-transport-https ca-certificates curl`


# INSTALL KUBEADM, KUBELET, KUBECTL [FOR BOTH NODES]

You can use this link or continue following next steps [https://v1-28.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl]

`sudo apt-get update`

`sudo apt-get install -y apt-transport-https ca-certificates curl gpg`

`curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg`

`echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list`

`sudo apt-get update`

`sudo apt-get install -y kubelet kubeadm kubectl`

`sudo apt-mark hold kubelet kubeadm kubectl`


# INSTALL CALICO AND INIT NOE [FOR HEAD NODE]

You can use this link or continue following next steps [https://v1-28.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl]

`sudo kubeadm init --pod-network-cidr=192.168.0.0/16`

_____________
After it you will get join command at the end. Save it. It should return:

`kubeadm join 192.168.0.4:6443 --token owbfiz.eokqnp6nh3vuliry \
        --discovery-token-ca-cert-hash sha256:d4c94ba0f1bd354dcdcc03f757250d030bbee19ab5369566dc366bede131b760`
_____________

`mkdir -p $HOME/.kube`

`sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`

`sudo chown $(id -u):$(id -g) $HOME/.kube/config`

`kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.1/manifests/tigera-operator.yaml`

`kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.1/manifests/custom-resources.yaml`

`watch kubectl get pods -n calico-system` - Wait until each pod has the `STATUS` of `Running`

`kubectl taint nodes --all node-role.kubernetes.io/control-plane-`

`kubectl get nodes -o wide`

It should return:

```
  NAME              STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
  <your-hostname>   Ready    master   52m   v1.12.2   10.128.0.28   <none>        Ubuntu 18.04.1 LTS   4.15.0-1023-gcp   docker://18.6.1

```

### If your node have status Not-Ready look at `TROUBLESHOOTING` section


# JOIN WOROKER NODE [FOR WORKER NODE]

Command that you have saved while init head node in previous step

`kubeadm join 192.168.0.4:6443 --token owbfiz.eokqnp6nh3vuliry \
        --discovery-token-ca-cert-hash sha256:d4c94ba0f1bd354dcdcc03f757250d030bbee19ab5369566dc366bede131b760`

# TROUBLESHOOTING [FOR BOTH NODES]

If nodes can't change their state for `Ready` try to do install cni again

`wget https://github.com/containernetworking/plugins/releases/download/v0.8.2/cni-plugins-linux-amd64-v0.8.2.tgz`

`mkdir cni`

`tar cni-plugins-linux-amd64-v0.8.2.tgz -C cni`

`mkdir -p /opt/cni/bin`

`cp ./cni/* /opt/cni/bin/`

`mkdir -p /etc/cni/net.d/`

`cat >/etc/cni/net.d/bridge.conf <<EOF
{
  "cniVersion": "0.3.1",
  "name": "containerd-net",
  "type": "bridge",
  "bridge": "cni0",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "subnet": "10.88.0.0/16",
    "routes": [
      { "dst": "0.0.0.0/0" }
    ]
  }
}
EOF`

# Wait for few minutes, status should be `Ready`
