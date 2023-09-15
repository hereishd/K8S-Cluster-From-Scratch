# Deploy NGINX as reverse proxy


## Before you Begin
We already have our Cluster running on VM's, I also recommend you to spin up a VM for your rever proxy.</br>
You could be tempted of running your nginx in a container but I do not recommend this approach since the DNS mapping from your host file can not be detected by your NGINX configurations. There is probably a way to make it work by tweaking some settings but anyhow, in a working environment, you will get a separate server with NGINX, hence this is a good exercise.<br/>
This being said, to be able to follow this doc, you will need a new VM running Ubuntu (I used version 22.04) provisionned with 2CPU's and 4GB of RAM. I allocated 40GB of disk space but this is up to you.<br/>
I assume you already have your ingress controller installed (here I am using NGINX ingres controller) but if this is not the case, no worries, we will configure it for running the ingress-controller's service as type NodePort via Helm. This way, the command will or configure/upgrade the release you already have for you or make a fresh install in the other case.

## Why NodePort
My setup here makes use of NodePort service because I am currently on a mandate that uses an external Cirtix Load balancer that we need to point to an NGINX reverse proxy (that we need to setup) in order to deliver the demanded cluster Ingress Resources.

## Configuring/Installing the NGINX Ingress Controller with NodePort service
To set the service on NodePort at installation or on upgrade we need to add the ```--set controller.service.type=NodePort``` flag in our command.
```
$ helm upgrade -i ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--namespace ingress-nginx \
--set controller.service.type=NodePort
```
We can now check our installation with 
```
$ kubectl get all -n ingress-nginx
```
You can see that the ```service/ingress-nginx-controller``` is of type NodePort.<br/><br/>
You can also test that it is pointing to your ingress-controller via the port that has been assigned. For this select the appropiate worker's cluster IP (since the controller is on the worker node) and add the mapping port from your ingress-nginx-controller service (ex: 192.168.100.15:31647). Since we did not declare any ingress resource we should land on the NGINX 404 not found page.<br/><br/>
Note that doing it this way will always have our NodePort changing when stoping and restarting the controller (evry time we reboot the VMs). To set our NodePort, we will edit our manifest file.
* Obtain your manifest file
```
helm template ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--namespace ingress-nginx \
--set controller.service.type=NodePort > manifest.yaml
```
* Edit your manifest.yaml
In our manifest file search for NodePort. In the Service where you find your NodePort, add the node port field and define your ports. It will then look like
```
type: NodePort
  ipFamilyPolicy: SingleStack
  ipFamilies: 
    - IPv4
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
      appProtocol: http
      nodePort: 30100
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
      appProtocol: https
      nodePort: 30200
```
* Apply the manifest
```
$ kubectl apply -f manifest.yaml
```
* Check our NodePorts
```
$ kubectl get svc -n ingress-nginx
```
You can now see that our node ports have been asigned the numbers we gave them and this will be the case every time we reboot our VMs.
## Create an Ingress
Now, I will deploy a sample application with an Ingress.

## Setting up the NGINX reverse proxy
* Install NGINX on  The VM
```
$ sudo apt install nginx
```
* Disable the default virtual host
```
$ sudo unlink /etc/nginx/sites-enabled/default
```
* Create the NGINX reverse proxy configuration
For this, We create a file under ```etc/nginx/sites-available```. I chose to name it ```reverse-proxy.conf```.<br/>
Here, we will define **upstream** blocks that are each used to define a cluster that you can proxy requests to. In this block, we are going to pass the <IP:PORT> of our NGINX Ingress Controller. Note that if your cluster has an internal Load Balancer, we just need to put it's EXTERNAL_IP.<br/>
We also define a **location** block where we are going to define where to redirect the request via **proxy_pass**. We are also going to set the headers for **proxy_set_headers** in order to successfully direct our request.<br/>
We will also make use og NGINX variables **$host** to grab the host from our request and assign it to the **upstream** variable. This way, we are going to name our upstream blocks the same way as our host request and pass them to the proxy_pass instruction. <br/>
The **listen** instruction defines the port that NGINX listens on.<br/>Our file will look like this:
```
upstream example.cluster1.com {
	server <Ingress-Controller's IP:PORT>;
}

server {

	listen 80;

	set $upstream $host;

	location / {
		proxy_pass http://$upstream;
		proxy_set_header Host $host;
		proxy_set_header X-Forwarded-For $remote_addr;
	}
}
```
* Link directives to sites-enabeled
Now, we activate the directives by linking them to ```/sites-enabled/```
```
$ sudo ln -s /etc/nginx/sites-available/reverse-proxy.conf /etc/nginx/sites-enabled/reverse-proxy.conf
```
* Apply ou settings
To apply our setting we have the option of restarting our NGINX service
```
$ sudo systemctl restard nginx
```
Or we can reload the settings 
```
$ nginx -s reload
```
* Debug in case of error
When reload or settings or restarting our service, we can get an error if NGINX has a problem with our config file. By running the following command
```
$ nginx -t
```
We can get an output of the error code/reason that can help uf fix the error. 


## Additional Notes
If we decide to eventually spin up more clusters, In order to add them to our reverse proxy we will only have to add more upstream blocks:
```
upstream example.cluster2.com {
    server <Ingress-Controller's IP:PORT>;
}
```
In case ou Ingress Controler is set to node balance, we only need to remove the PORT from the IP:PORT because 80 is the port by default.
```
upstream example.cluster3LoadBalncer.com {
    server <Ingress-Controller's IP>;
}
```

## References
* [NGINX  doc] (https://nginx.org/en/)
* [NGINX ngx_http_upstream_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html)
