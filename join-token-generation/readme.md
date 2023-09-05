# Join worker nodes by creating a new token

A token is required when joining a new worker node to the Kubernetes cluster. When you bootstrap a cluster with kubeadm, a token is generated which expires after 24 hours. If the join token is expired and you want to join a 
new node, you will need to create a new token.

## Generating a new token

* Check if you have a valid token
```
$ kubeadm token list
```
* If the token is expired, generate a new one
```
$ sudo kubeadm token create
```
* Grab the token generated
```
$ kubeadm token list
```
## Get the Discovery Token CA cert Hash (from Master node)
```
$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```
* Get API Server Advertise address
```
$ kubectl cluster-info
```
## Joining some new Kubernetes Worker Node to your Cluster

The kubeadm 'join' command is used to bootstrap a Kubernetes worker node or an additional control plane node, and join it to the cluster. The command syntax for joining a worker node to cluster.
<br/>
The common flags required are ```--token``` and ```--discovery-token-ca-cert-hash```
<br/>So our complete join command will have the format:
```
sudo kubeadm join \
  192.168.XXX.XXX:6443 \
  --token $TOKEN \
  --discovery-token-ca-cert-hash sha256:$CA_CERT_HASH
```