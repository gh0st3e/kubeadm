# kubeadm

Step#1
Install client-tools

1) install golang https://go.dev/doc/install

Step#2
Prepare kubeadm,kubelete,kubectl 
Common guide (https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
1) install Docker https://docs.docker.com/engine/install/ubuntu/
2) Docker postinstall https://docs.docker.com/engine/install/linux-postinstall/ [optional]
3) install kubelet kubectl kubeadm https://v1-28.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl

Step#3
Init Node
1) init node https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart
  1.1) if cri doesn't work
     `rm /etc/containerd/config.toml`
     `systemctl restart containerd`
     `kubeadm init`
2)  


---------------------------------------------------------------------------------------------------

KUBERNETES 1.26
CONTAINERD 1.6.16
UBUNTU 22.04

### ALL: 

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

`nano /etc/containerd/config.toml`   <<<<<<<<<<< manually edit and change systemdCgroup to true

`systemctl restart containerd`

`swapoff -a`  <<<<<<<< just disable it in /etc/fstab instead

`apt-get update`

`apt-get install -y apt-transport-https ca-certificates curl`


`curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg`

`echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list` <<<<< better look in k8s official web site


`apt-get update`

`reboot`

`apt-get install -y kubelet=1.26.1-00 kubeadm=1.26.1-00 kubectl=1.26.1-00`

`apt-mark hold kubelet kubeadm kubectl`

# check swap config, ensure swap is 0
`free -m`


### ONLY ON CONTROL NODE .. control plane install:
`kubeadm init --pod-network-cidr 10.10.0.0/16 --kubernetes-version 1.26.1 --node-name k8s-control`


# add Calico 3.25 CNI 
### https://docs.tigera.io/calico/3.25/getting-started/kubernetes/quickstart
`kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml`

`wget https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml`

`vi custom-resources.yaml` <<<<<< edit the CIDR for pods if its custom

`kubectl apply -f custom-resources.yaml`

# get worker node commands to run to join additional nodes into cluster

`kubeadm token create --print-join-command`


## ONLY ON WORKER nodes
Run the command from the token create output above
