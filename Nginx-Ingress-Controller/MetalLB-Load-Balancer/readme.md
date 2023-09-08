# Deploy MetalLB Load Balancer on Kubernetes Cluster

MetalLB is a pure software solution that provides a network load-balancer implementation for Kubernetes clusters that are not deployed in supported cloud provider using standard routing protocols. By installing MetalLB solution, you effectively get LoadBalancer Services within your Kubernetes cluster.

## Requirements
* A Kubernetes cluster on version 1.13.0 or later. The cluster should not have another network load-balancing functionality.
* A cluster network configuration that can coexist with MetalLB.
* Availability of IPv4 addresses that MetalLB will assign to LoadBalancer services when requested.

## Deploy MetalLB Load Balancer
* Update system
```
$sudo apt update
```
* Get the latest MetalLB release tag
```
$ MetalLB_RTAG=$(curl -s https://api.github.com/repos/metallb/metallb/releases/latest|grep tag_name|cut -d '"' -f 4|sed 's/v//')
```
* Create a directory and download the manifest
```
$ mkdir metallb
$ cd metallb
$ wget https://raw.githubusercontent.com/metallb/metallb/v$MetalLB_RTAG/config/manifests/metallb-native.yaml
```
* Install MetalLB Load Balancer
Deploy MetalLB to your Kubernetes cluster, under the metallb-system namespace.
```
$ kubectl apply -f metallb-native.yaml
```
Wait till everythis is running.
```
$ kubectl get all -n metallb-system
```
## Setting up the the configs
The installation manifest does not include a configuration file required to use MetalLB. All MetalLB components are started, but will remain in idle state until you finish the necessary configurations. 
* Create Load Balancer services Pool of IP Addresses
MetalLB needs a pool of IP addresses to assign to the services when it gets such request. We have to instruct MetalLB to do so via the IPAddressPool CR.<br/>
Let’s create a file with configurations for the IPs that MetalLB uses to assign IPs to services.<br/>
  * Create a metallb-config.yaml file
  * Add the following content to the file
  ```
  apiVersion: v1
  kind: ConfigMap
  metadata:
    namespace: metallb-system
    name: config
  data:
    config: |
      address-pools:
      - name: addresspool-sample1
        protocol: layer2
        addresses:
        - 172.18.1.1-172.18.1.16
  ```