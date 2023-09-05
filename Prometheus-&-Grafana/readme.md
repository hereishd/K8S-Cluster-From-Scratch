# Setup Prometheus & Grafana

This guide is created with the intention of guiding us to Setup Prometheus and Grafana on Kubernetes using prometheus-operator.<br/>
Prometheus is a full fledged solution that enables access to advanced metrics capabilities in Kubernetes. The metrics are collected in a time interval of 30 seconds, this is a default settings. The information collected include resources such as Memory, CPU, Disk Performance and Network IO as well as R/W rates. By default the metrics are exposed on your cluster for up to a period of 14 days, but the settings can be adjusted to suit your environment.<br/>
Grafana is used for analytics and interactive visualization of metrics that’s collected and stored in Prometheus database. You can create custom charts, graphs, and alerts for Kubernetes cluster, with Prometheus being data source. In this guide we will perform installation of both Prometheus and Grafana on a Kubernetes Cluster. For this setup kubectl configuration is required, with Cluster Admin role binding.<br/>
<br/>
To get a complete an entire monitoring stack we will use kube-prometheus project which includes Prometheus Operator among its components. The kube-prometheus stack is meant for cluster monitoring and is pre-configured to collect metrics from all Kubernetes components, with a default set of dashboards and alerting rules.

## Deploy Prometheus / Grafana Monitoring Stack on Kubernetes

* Clone the kube-prometheus project
```
$ git clone https://github.com/prometheus-operator/kube-prometheus.git
```
* Navigate to the kube-prometheus directory
```
cd kube-prometheus
```
* Create a monitoring namespace and required CustomResourceDefinitions
```
$ kubectl create -f manifests/setup
```
* Deploy the Prometheus Monitoring Stack on Kubernetes
```
kubectl create -f manifests/
```
&nbsp; Give it few seconds and the pods should start coming online. This can be checked ```$ kubectl get pods -n monitoring -w```<br/>
&nbsp; To check all the services created run ```$ kubectl get svc -n monitoring```
## Accessing Prometheus, Grafana, and Alertmanager dashboards
We now have the monitoring stack deployed. There are two ways to access the dashboards of Grafana, Prometheus and Alertmanager:
  * Accessing Prometheus UI and Grafana dashboards using kubectl proxy
    grafana: ```$ kubectl --namespace monitoring port-forward svc/grafana 3000```, prometheus: ```kubectl --namespace monitoring port-forward svc/prometheus-k8s 9090```, alertmanager: ```kubectl --namespace monitoring port-forward svc/alertmanager-main 9093```
    <br/>*The default grafana credentials are admin/admin*
  * Accessing Prometheus UI and Grafana dashboard using NodePort / LoadBalancer
    To access Prometheus, Grafana, and Alertmanager dashboards using one of the worker nodes IP address and a port you’ve to edit the services and set the type to NodePort.
    You need a Load Balancer implementation in your cluster to use service type LoadBalancer
    *The Node Port method is only recommended for local clusters not exposed to the internet. The basic reason for this is insecurity of Prometheus/Alertmanager services.*
    // TODO: Explain this step.
## Storing data in Persistent volume (Optional)
By default, the operator configures Pods to store data on emptyDir volumes which aren’t persisted when the Pods are redeployed. To maintain data across deployments and version upgrades, you can configure persistent storage for Prometheus, Alertmanager and ThanosRuler resources.
// TODO: Document method.