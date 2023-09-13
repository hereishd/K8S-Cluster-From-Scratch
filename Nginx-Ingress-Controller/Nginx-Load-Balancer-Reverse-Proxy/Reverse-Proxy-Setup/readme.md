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
You can also test that it is pointing to your ingress-controller via the port that has been assigned. For this select the appropiate worker's cluster IP (since the controller is on the worker node) and add the mapping port from your ingress-nginx-controller service (ex: 192.168.100.15:31647). Since we did not declare any ingress resource we should land on the NGINX 404 not found page.

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
 For this, We create a file under ```etc/nginx/sites-available```. I chose to name it ```reverse-proxy.conf```.