# K8S Cluster From Scratch (VirtualBox, Ubuntu 22.04LTS)

This is my personal documentation containing the steps for building a Kubernetes Cluster from scratch on VirtualBox with Ubuntu 22.04LTS.

## Before you begin

* [VirtualBox](https://www.virtualbox.org/wiki/Downloads) installed on your machine
* A Ubuntu [22.04LTS](https://releases.ubuntu.com/jammy/) iso image 

*Please note that you will need enough ressources to at least provide 2 VMs with 2GB of RAM or more and 2CPUs or more each.*

## Initial Setup

We first need to create our VMs: a VM for our Control Plane Node and one (or more) for our Worker Nodes. Each machine should be provisionned with:

* Ubuntu 22.04LTS OS
* At least 2GB of RAM
* At least 2CPUs.
* At least 25GB of disk space.

*I also recommend to check the 'Guest Additions' from the installation menu as it will allow us to enable the shared clipboard from host to guest.*

## Setup a Host-Only Network on Virtualbox

## Ubuntu Setup

* First add your user to sudo
```
$ su
$ usermod -aG sudo $USERNAME
$ reboot
```
* Update & Upgrade Ubuntu
```
$ sudo apt update
$ sudo apt -y full-upgrade
$ [ -f /var/run/reboot-required ] && sudo reboot -f
```
* Install tools
```
$ sudo apt install curl apt-transport-https -y
$ sudo apt install net-tools
```
* Disable firewall
```
$ sudo uwf disable
```
* Disable swap
```
$ swapoff -a
$ sudo sed -i.bak -r 's/(.+ swap .+)/#\1/' /etc/fstab
```
&nbsp;*You can confirm these settings are correct with:*```$ free -h```
* On each VM, get your host static IP (should be the IP of adapter enp0s8).
```
$ ifconfig
```
&nbsp;Note the IP from enp0s8
<br/><br/>
&nbsp;Alternatively, you can get the IP from:
```
ip addr show | grep enp0s8
```
&nbsp; *keep the IP (192.168.XXX.XXX) and takeout the range '/24'*
* Get each VM, get your hostname
```
$ hostname
```
* Edit each VM's hosts file
```
$ sudo nano /etc/hosts
```
&nbsp;For every VM, add the IP and hostname of the other VMs it will communicate with.
```
ex: kmaster@kamaster:~$ nano /etc/hosts

127.0.0.1       localhost
127.0.0.1       kmaster.myguest.virtualbox.org kmaster
192.168.XXX.XXX kworker1  <= For every node, add IP and hostname
```
## Install Kubelet, Kubeadm, Kubectl
```
$ curl -fsSL  https://packages.cloud.google.com/apt/doc/apt-key.gpg|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/k8s.gpg
$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt update
$ sudo apt install wget git kubelet kubeadm kubectl -y
$ sudo apt-mark hold kubelet kubeadm kubectl
```
&nbsp;*You can validate the installation with:* ```$ kubectl version --client``` *&* ```$ kubeadm version```
## Enable Kernel modules and configure systemctl
* Enable kernel modules
```
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```
* Configure sysctl
```
$ sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  net.ipv4.ip_forward = 1
  EOF
```
* Reload sysctl
```
$ sudo sysctl --system
```
## Install a Container Runtime (in our case, CRI-O)
* Configure the persistent loading of modules
```
$ sudo tee /etc/modules-load.d/k8s.conf <<EOF
  overlay
  br_netfilter
  EOF
```
* Ensure you load modules
```
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```
* Set up required sysctl params
```
$ sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  net.ipv4.ip_forward = 1
  EOF
```
* Reload sysctl
```
$ sudo sysctl --system
```
* Add the CRI-O repo
```
$ su
$ export OS="xUbuntu_22.04"
$ export VERSION=1.26
$ echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
$ echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
$ curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | apt-key add -
$ curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key add -
```
&nbsp; *For the OS & VERSION variables you can refer to [the CRI-O documentation](https://github.com/cri-o/cri-o/blob/main/install.md#readme)*
* Install CRI-O
```
$ sudo apt update
$ sudo apt install cri-o cri-o-runc
```
* Start and enable the Service
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart crio
$ sudo systemctl enable crio
$ systemctl status crio
```
## Initiate the Controle Plane Node (Master Node Only)
* Enable the Kubelet service
```
$ sudo systemctl enable kubelet
```
* Pull the container images *(This is not required as they will be pulled on 'kubeadmin init')*
```
$ sudo kubeadm config images pull --cri-socket /var/run/crio/crio.sock
```
* Initiate the Master Node with Kubeadm
```
$ export MASTER_IP=<your-master-node-static-IP>
$ sudo kubeadm init --apiserver-advertise-address=$MASTER_IP --apiserver-cert-extra-sans=$MASTER_IP --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///var/run/crio/crio.sock --v=5
```
&nbsp;*Don't forget to set the MASTER_IP with your master node's static IP*<br/>
&nbsp;*You can find a full list of options to pass to 'kubeadm init' and explainations [here](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/).*<br/>
&nbsp;*I recommand you to save the output of the 'init' command as it contains your 'join' command infos.*
* Setup you kube config file
```
$ mkdir -p $HOME/.kube
$ sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
* Check the cluster status
```
$ kubectl cluster-info
```
## Join the Worker Nodes (Worker Nodes Only)
With the control plane ready you can add worker nodes to the cluster for running scheduled workloads.<br/>
The join command that was given on the 'kubeadm init' output is used to add worker nodes to the cluster.
```
ex: kubeadm init output

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.XXX.XXX:6443 --token 0mtvva.1snrqiu4snaivpoi \
  --discovery-token-ca-cert-hash sha256:be9e86388bbeffeae73dd6f27375dbb5b25a33e8a316dfa8a4a4839b357360b6
```
* Execute the join command from the output on each worker node
```
kubeadm join 192.168.XXX.XXX:6443 --token 0mtvva.1snrqiu4snaivpoi \
  --discovery-token-ca-cert-hash sha256:be9e86388bbeffeae73dd6f27375dbb5b25a33e8a316dfa8a4a4839b357360b6
```
&nbsp;*Note that the token expires within 24h. To add more nodes in the future you will need to generate new tokens.*
## Set Kubelet Node IPs
The setup is now almost done. If you go back to the master node and check the cluster info : ```$ kubectl get node``` you will see all your nodes and they should be in a Ready state.
<br/>
If you run the commande with the wide option: ```$kubectl get nodes - wide``` you will realise that all the nodes 'INTERNAL IP' have the same. We need to change this on each node.
* Set Node Ip in the kubelet
```
$ sudo nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
&nbsp;Add the ```--node-ip=192.168.XXX.XXX``` flag to KUBELET_CONFIG_ARGS
* Restart Kubelet
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```
&nbsp; You can now check that your changes have been applied
```
$ kubectl get nodes -o wide
```

## References
* [VirtualBox Networking](https://www.virtualbox.org/manual/ch06.html)
* [Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
* [Install CRI-O](https://github.com/cri-o/cri-o/blob/main/install.md#readme)
* [Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
