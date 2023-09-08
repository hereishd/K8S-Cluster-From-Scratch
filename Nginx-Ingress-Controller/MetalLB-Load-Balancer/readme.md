# Deploy MetalLB Load Balancer on Kubernetes Cluster

MetalLB is a pure software solution that provides a network load-balancer implementation for Kubernetes clusters that are not deployed in supported cloud provider using standard routing protocols. By installing MetalLB solution, you effectively get LoadBalancer Services within your Kubernetes cluster.

## Requirements
* A Kubernetes cluster on version 1.13.0 or later. The cluster should not have another network load-balancing functionality.
* A cluster network configuration that can coexist with MetalLB.
* Availability of IPv4 addresses that MetalLB will assign to LoadBalancer services when requested.

## Preparation
If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.<br/>
*Note, you don’t need this if you’re using kube-router as service-proxy because it is enabling strict ARP by default.*<br/><br/>

You can achieve this by editing kube-proxy config in current cluster:
```
$ kubectl edit configmap -n kube-system kube-proxy
```
and set:
```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```
## Deploy MetalLB Load Balancer
* Update system
```
$sudo apt update
```
* Install MeltalLb
```
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
```
## Setting up the the configs
The installation manifest does not include a configuration file required to use MetalLB. All MetalLB components are started, but will remain in idle state until you finish the necessary configurations. 
* Create Load Balancer services Pool of IP Addresses<br/>
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
  *Note that for the addresses at the bottom of the file, we need to set a range of IP addresses that are available on our system for MetalLb to hand out to our services*
    * Create the Configmap
    ```
    $ kubectl apply -f metallb-config.yaml
    ```