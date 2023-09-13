# Exposing Prometheus Stack (kube-prometheus)

Depending on how you installed your Prometheus Stack, there is 2 different ways of accomplishing these tasks.<br/>
Here, I will explain the method for a kube-prometheus installed via manifest to your cluster.<br/>
In case you stack has been installed with Helm, please follow my documentation on this matter.

## Before you begin

Apart from a running Kubernetes cluster with a running kube-prometheus stack, a Kubernetes Ingress controller must be installed and functional. This guide was tested with the nginx-ingress-controller.

## Setting up your Ingress

For each of your services, we will setup an Ingress as follow, mapping the desired host to our service. Please make sure to select the right services and ports.<br/>
* Declare your Ingress
So first, let's create our manifest file named ```monitoring-ingress.yaml``` and append this content to it.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus
  namespace: monitoring
spec:
  ingressClassName: nginx
  rules:
  - host: prometheus.cluster1.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-k8s
            port:
              number: 9090

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alertmanager
  namespace: monitoring
spec:
  ingressClassName: nginx
  rules:
  - host: alertmanager.cluster1.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: alertmanager-main
            port:
              number: 9093

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
spec:
  ingressClassName: nginx
  rules:
  - host: grafana.cluster1.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 3000
```
* Create the resources
```
$ kubectl apply -f monitoring-ingress.yaml
```
* Set your hosts
Do not forget, as we are working with VM's and fake hosts you need to edit your hosts file
```
$ sudo nano /etc/hosts
```
And not add a line for each of your Ingress mapping your Ingress controller's EXTERNAL_IP to each of your Ingress's hosts.

## Additional notes & Reference

The [official documentation](https://github.com/prometheus-operator/kube-prometheus/blob/main/docs/customizations/exposing-prometheus-alertmanager-grafana-ingress.md) sets this up making use of a basic authentication. It uses a secret and ```htpasswd```. You can always take a look if you are interested.