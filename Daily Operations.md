# Operations

## Start / Stop RKE2 Services
The only common command for starting and stopping services in kubernetes cluster are as follows:

For Master Node:

Start:
```bash
systemctl start rke2-server
```

Stop:
```bash
systemctl stop rke2-server
```

For Worker Node:

Start:
```bash
systemctl start rke2-agent
```

Stop:
```bash
systemctl stop rke2-agent
```

## Checking the Status of Nodes, Pods and Deployment (Run these command on haproxy node for cluster management purposes):

Check Node Status:
```bash
kubectl get nodes
```

Check Pod Status:
```bash
#in specific namespace
kubectl get pods -n yourownnamespace
```
```bash
#In all namespace
kubectl get pods -A
```

Check Deployment:
```bash
#In specific namespace:
kubectl get deployments -n yourownnamespace
```
```bash
#In all namespace
kubectl get deployments -A
```
## Pods Scaling
Scaling is done automatically using horizontal pod autoscalers which are set by the developers in their pipeline. The treshold to scale can be set accordingly.

## Adding and Removing Nodes

## For Master Node

### Adding master node
#### 1. Install RKE2:
```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=server sudo sh -
```
### 2. Create the following file (if its not created) `/etc/rancher/rke2/config.yaml`
```bash
vim /etc/rancher/rke2/config.yaml
```
Add the following content into the file:
```bash
server: https://keepalived-ip:9345
token: yourownsecuritytokensameasfirstmasternode
tls-san:
  - keepalived-proxy-ip
  - keepalived-proxy-name
```
### 3. Start RKE2
```bash
systemctl enable rke2-server
systemctl start rke2-server
```

## For Worker Node
### Adding worker node
#### 1. Install RKE2:
```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=agent sudo sh -
```
For rocky linux, you must dnf install the rke2-agent
```bash
dnf install rke2-agent
```
### 2. Create the following file (if its not created) `/etc/rancher/rke2/config.yaml`
```bash
vim /etc/rancher/rke2/config.yaml
```
Add the following content into the file:
```bash
server: https://keepalived-ip:9345
token: yourownsecuritytokensameasfirstmasternode
tls-san:
  - keepalived-proxy-ip
  - keepalived-proxy-name
```
### 3. Start RKE2
```bash
systemctl enable rke2-agent
systemctl start rke2-agent
```
## Removing Nodes

### 1. Drain (Remove pods running on it) and remove the node from the cluster
Login to one of the haproxy node

a. List out all the nodes and get the node name
```bash
kubectl get nodes
```
b. Drain the node
```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```
c. Delete it from the cluster
```bash
kubectl delete node <node-name>
```

### 2. Stop RKE2 service on the removed node
For master node
```bash
sudo systemctl stop rke2-server
sudo systemctl disable rke2-server
```
For worker node
```bash
sudo systemctl stop rke2-agent
sudo systemctl disable rke2-agent
```

### 3. Uninstall RKE2 from the removed node
```bash
sudo /usr/bin/rke2-uninstall.sh
```



