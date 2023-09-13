# Deploy NGINX as reverse proxy


## Before you Begin
We already have our Cluster running on VM's, I also recommend you to spin up a VM for your rever proxy.</br>
You could be tempted of running your nginx in a container but I do not recommend this approach since the DNS mapping from your host file can not be detected by your NGINX configurations. There is probably a way to make it work by tweaking some settings but anyhow, in a working environment, you will get a separate server with NGINX, hence this is a good exercise.<br/>
This being said, to be able to follow this doc, you will need a new VM running Ubuntu (I used version 22.04) provisionned with 2CPU's and 4GB of RAM. I allocated 40GB of disk space but this is up to you.<br/>
I assume you already have your ingress controller installed (here I am using NGINX ingres controller) but if this is not the case, no worries, we will configure it for running the ingress-controller's service as type NodePort via Helm. This way, the command will or configure/upgrade the release you already have for you or make a fresh install in the other case.

##  