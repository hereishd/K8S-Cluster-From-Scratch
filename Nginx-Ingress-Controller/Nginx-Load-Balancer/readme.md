# Deploy NGINX as Kubernetes Bare Metal Load Balancer

Nginx is a very popular web server that is used throughout many organizations worldwide. However, it can be also be used as a bare metal load balancer for Kubernetes workloads. It is an open-source solution, simple, easy to set up, and lightweight. It provides a great option for those looking for an easy load balancer solution in their home lab environment. Here, we will take a look at Kubernetes bare metal load balancer configuration with Nginx and see how this can be configured.

## Install NGINX in the home lab environment
Here I will discuss 2 options.
* Setting up NGINX in a new Ubuntu VM.
This is a good options since it can apparent to a real world scenario but, as a downside, it will consume more resources on you machine that is already running VMs for your cluster.
* Setting up NGINX in a docker image
Going this way, you will not configure the NGINX Load Balancer directly on a server but it is a lightweight solution that allows understanding of the concept.

## Option 1: Setting up NGINX in a new Ubuntu VM
* Create a new Ubuntu VM on virtualbox (here I used Ubuntu 22.04).
This machine does not need a lot of ressources. I gave it 4GB of RAM, 2CPUs and 25GB of disk space.
* Add your user to sudo
```
$ su
$ usermod -aG sudo $USERNAME
$ reboot
```
* Update and Upgrade the machine
```
$ sudo apt update
$ sudo apt -y full-upgrade
$ [ -f /var/run/reboot-required ] && sudo reboot -f
```
* Install NGINX
```
sudo apt-get install nginx -y
```
* Edit the NGINX config file
After you install Nginx, the work we will do will be in the /etc/nginx/nginx.conf file.
```
$ sudo nano /etc/nginx/nginx.conf
```
The file will have content like below
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

...
```
Here, you You will want to comment out the lines below the “events” (adding # at the beggining of each) or simply delete them.
* Add the load balance configuration
As an example of a simple way to load balance the API access to a master node, you can add the following code in the nginx.conf under the events section.
```
stream { 
  upstream k8s { 
    server 192.168.XXX.XXX:6443; 
  } 
  server { 
    listen 6443; 
    proxy_pass k8s; 
  } 
}
```
**Upstream** defines the target servers you will be accessing in your Kubernetes clusters and the ports</br>
**server** defines the port on your local Nginx host that is listening on a specific port<br/>
**proxy_pass** tells Nginx that you will be passing this traffic to the upstream servers you have configured


## Option 2: Setting up NGINX in a docker image  