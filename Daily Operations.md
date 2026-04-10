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
