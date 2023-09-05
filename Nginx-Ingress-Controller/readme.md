# NGINX Ingress Controller Setup (with Helm)

The standard way of exposing applications that are running on a set of Pods in a Kubernetes Cluster is by using service resource. Each Pod in Kubernetes has its own IP address, but a set of Pods can have a single DNS name. Kubernetes is able to load-balance traffic across the Pods without any modification in the application layer. A service, by default is assigned an IP address (sometimes called the “cluster IP“), which is used by the Service proxies. A Service is able to identify a set of Pods using label selectors.
<br/><br/>
Before explaining what is an Ingress Controller, we have to understand an Ingress Object in Kubernetes. From the [official Kubernetes documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/), an Ingress is defined as "An API object that manages external access to the services in a cluster, typically HTTP. Ingress may provide load balancing, SSL termination and name-based virtual hosting.".<br/>
An Ingress in Kubernetes exposes HTTP and HTTPS routes from outside the cluster to services running within the cluster. All the traffic routing is controlled by rules defined on the Ingress resource. An Ingress may be configured to: provide services with externally-reachable URLs / to load  balance traffic coming into cluster services / to terminate SSL TLS traffic / to provide name-based virtual hosting in Kubernetes.<br/>
An Ingress controller is what fulfils the Ingress, usually with a load balancer. Basically, it works to enact the rules you have set, usually using an HTTP or L7 load balancer. It ensures that your inputs to the Ingress Resource are followed consistently.<br/><br/>
Below is an example on how an Ingress sends all the client traffic to a Service in Kubernetes Cluster.</br>
![Schema](../img/NGINX_Controller_design.png)
<br/>
For the standard HTTP and HTTPS traffic, an Ingress Controller will be configured to listen on ports 80 and 443. It should bind to an IP address from which the cluster will receive traffic from. A Wildcard DNS record for the domain used for Ingress routes will point to the IP address(s) that Ingress controller listens on.<br/>
 You can choose from the [plenty of Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) available. Here, we will be using the [NGINX one from Kubernetes](https://github.com/kubernetes/ingress-nginx/).
<br/></br>
With all these basics on Kubernetes Services and Ingress, we can now plunge into the actual installation of NGINX Ingress Controller Kubernetes.


## Additional Notes
The deployment process varies depending on your Kubernetes setup. My Kubernetes uses the Bare-metal NGINX Ingress deployment guide. For other Kubernetes clusters including managed clusters refer to below guides:
* [minikube](https://kubernetes.github.io/ingress-nginx/deploy/#minikube)
* [microk8s](https://kubernetes.github.io/ingress-nginx/deploy/#microk8s)
* [AWS](https://kubernetes.github.io/ingress-nginx/deploy/#aws)
* [Azure](https://kubernetes.github.io/ingress-nginx/deploy/#azure)
* [GCE-GKE](https://kubernetes.github.io/ingress-nginx/deploy/#gce-gke)

If no above option is suitable for you, others are also available. Refer to the [official documentation](https://kubernetes.github.io/ingress-nginx/deploy/).