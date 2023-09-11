# Deploy MetalLB Load Balancer on Kubernetes Cluster (using Helm)

MetalLB is a pure software solution that provides a network load-balancer implementation for Kubernetes clusters that are not deployed in a supported cloud provider using standard routing protocols. By installing MetalLB solution, you effectively get LoadBalancer Services within your Kubernetes cluster.

## Requirements
* A Kubernetes cluster on version 1.13.0 or later. The cluster should not have another network load-balancing functionality.
* A cluster network configuration that can coexist with MetalLB.
* Availability of IPv4 addresses that MetalLB will assign to LoadBalancer services when requested.
* Helm installed.

*As you can see, for this method, you will need Helm installed. You can follow the 'Installing Helm' section from my [Helm documentation repository](https://github.com/hereishd/k8s_Tutorials/tree/main/Helm) or from the [original Helm website](https://helm.sh/docs/intro/install/).*

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
## Deploy MetalLB Load Balancer (using Helm)
* Update system
```
$sudo apt update
```
* Create the namespace
```
$ kubectl create ns metallb-system
```
* Download Helm MeltalLb repository
```
$ helm repo add metallb https://metallb.github.io/metallb
$ helm repo update
```
* Install MetalLB 
```
$ helm install metallb metallb/metallb
```
## Setting up the the configs
The installation manifest does not include a configuration file required to use MetalLB. All MetalLB components are started, but will remain in idle state until you finish the necessary configurations. In the previous versions, MetalLB was configured using a ConfigMap. This is no longer the case and now make use of CRs.
* Create Load Balancer services Pool of IP Addresses<br/>
MetalLB needs a pool of IP addresses to assign to the services when it gets such request. We have to instruct MetalLB to do so via the IPAddressPool CR.<br/>
Let’s create a file with configurations for the IPs that MetalLB uses to assign IPs to services.<br/>
  * Create a ```ipaddress_pools.yaml``` file
  * Add the following content to the file
  ```
  apiVersion: metallb.io/v1beta1
  kind: IPAddressPool
  metadata:
    name: production
    namespace: metallb-system
  spec:
    addresses:
    - 192.168.1.30-192.168.1.50
  ```
  *Note that for the addresses at the bottom of the file, we need to set a range of IP addresses that are available on our system for MetalLb to hand out to our services*<br/><br/>
  * Announce service IPs after creation by creating a l2advert.yaml 
  * Add the following content to the file.
  ```
  apiVersion: metallb.io/v1beta1
  kind: L2Advertisement
  metadata:
    name: l2-advert
    namespace: metallb-system
  ```
## Verify your installation
To verify that your installation was a success check the ingress-controller service
```
$ kubectl get svc -n ingress-nginx
```
You should now see that your ingress controller's LoadBalancer EXTERNAL-IP has been assigned.

## Mapping DNS name for Nginx Ingresses to LB IP
We can create domain name, preferably wildcard for use when creating Ingress routes in Kubernetes.
<br/><br/>
You can now go back to finish the your [Ingress Controller Setup](https://github.com/hereishd/K8S-From-Scratch/tree/main/Nginx-Ingress-Controller)