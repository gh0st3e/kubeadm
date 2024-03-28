# kubeadm

Step#1
Install client-tools

1) install golang https://go.dev/doc/install
2) install CFSSL https://github.com/cloudflare/cfssl
   `chmod +x cfssl cfssljson`
   `sudo mv cfssl cfssljson /usr/local/bin/`

Step#2
Prepare kubeadm,kubelete,kubectl 
Common guide (https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
1) install Docker https://docs.docker.com/engine/install/ubuntu/
2) Docker postinstall https://docs.docker.com/engine/install/linux-postinstall/ [optional]
3) Install cri-dockerd https://www.mirantis.com/blog/how-to-install-cri-dockerd-and-migrate-nodes-from-dockershim [optional]
5) install kubelet kubectl kubeadm https://v1-28.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl

Step#3
Init Node
1) init node https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart
  1.1) if cri doesn't work
     `rm /etc/containerd/config.toml`
     `systemctl restart containerd`
     `kubeadm init`
2)  
