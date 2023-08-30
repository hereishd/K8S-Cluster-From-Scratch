# K8S Cluster From Scratch (VirtualBox, Ubuntu 22.04LTS)

This is my personal documentation containing the steps for building a Kubernetes Cluster from scratch on VirtualBox with Ubuntu 22.04LTS.

## Before you begin

* [VirtualBox](https://www.virtualbox.org/wiki/Downloads) installed on your machine
* A Ubuntu [22.04LTS](https://releases.ubuntu.com/jammy/) iso image 

*Please note that you will need enough ressources to at least provide 2 Vms with 2GB of RAM or more and 2CPUs or more each.*

## Initial Setup

We first need to create our VMs: a VM for our Control Plane Node and one (or more) for our Worker Nodes. Each machine should be provisionned with:

* Ubuntu 22.04LTS OS
* At least 2GB of RAM
* At least 2CPUs.
* At least 25GB of disk space.

*I also recommend to check the 'Guest Additions' from the installation menu as it will allow us to enable the shared clipboard from host to guest.*

## Setup a Host-Only Network on Virtualbox

## Ubuntu Setup

## Insatll Kubelet, Kubeadm, Kubectl

## Enable Kernel modules and configure systemctl

## Install a Container Runtime (CRI-O)

## Initiate the Controle Plane Node (Master Node Only)

## Join the Worker Nodes (Worker Nodes Only)

## Set Kubelet Node IPs