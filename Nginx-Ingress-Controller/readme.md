# Nginx Ingress Controller Setup (with Helm)

The standard way of exposing applications that are running on a set of Pods in a Kubernetes Cluster is by using service resource. Each Pod in Kubernetes has its own IP address, but a set of Pods can have a single DNS name. Kubernetes is able to load-balance traffic across the Pods without any modification in the application layer. A service, by default is assigned an IP address (sometimes called the “cluster IP“), which is used by the Service proxies. A Service is able to identify a set of Pods using label selectors.
<br/><br/>
Before explaining what is an Ingress Controller, we have to understand an Ingress Object in Kubernetes. From the official Kubernetes documentation, an Ingress is defined as "An API object that manages external access to the services in a cluster, typically HTTP. Ingress may provide load balancing, SSL termination and name-based virtual hosting.".<br/>
An Ingress in Kubernetes exposes HTTP and HTTPS routes from outside the cluster to services running within the cluster. All the traffic routing is controlled by rules defined on the Ingress resource. An Ingress may be configured to provide services with externally-reachable URLs / to load  balance traffic coming into cluster services / to terminate SSL TLS traffic / to provide name-based virtual hosting in Kubernetes.<br/>
An Ingress controller is what fulfils the Ingress, usually with a load balancer.<br/>
Below is an example on how an Ingress sends all the client traffic to a Service in Kubernetes Cluster.</br>
![Schema](../img/NGINX_Controller_design/png)