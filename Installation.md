# Installation Guide 

# a. HAProxy & Keepalived Setup
Perform these steps on both HAProxy nodes

## HAProxy Installation
```bash
# For Ubuntu
sudo apt update && sudo apt install haproxy -y
```

### 1. Create `/etc/haproxy/haproxy.cfg` with the following configuration:
```bash
vim /etc/haproxy/haproxy.cfg
```
`/etc/haproxy/haproxy.cfg` content
```bash
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode tcp
    option tcplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000

frontend kubernetes-api
    bind *:9345  # RKE2 supervisor port
    default_backend kubernetes-masters

backend kubernetes-masters
    balance roundrobin
    option tcp-check
    server master1 10.240.5.167:9345 check #masternode1
    server master2 10.240.5.168:9345 check #masternode2
    server master3 10.240.5.169:9345 check #masternode3

frontend kubernetes-api-6443
    bind *:6443  # Kubernetes API server port
    default_backend kubernetes-api-servers

backend kubernetes-api-servers
    balance roundrobin
    option tcp-check
    server master1 10.240.5.167:6443 check #masternode1
    server master2 10.240.5.168:6443 check #masternode2
    server master3 10.240.5.169:6443 check #masternode3

#---------------------------------------------------------------------
# HTTP Ingress (port 80)
#---------------------------------------------------------------------
frontend ingress_http_frontend
    bind *:80
    mode tcp
    option tcplog
    default_backend ingress_http_backend

backend ingress_http_backend
    mode tcp
    balance roundrobin
    option tcp-check
    # Point to nodes where Ingress Controller runs
    # If Ingress runs on masters:
    server master1 10.240.5.167:80 check fall 3 rise 2
    server master2 10.240.5.168:80 check fall 3 rise 2
    server master3 10.240.5.169:80 check fall 3 rise 2
    server worker1 10.240.5.172:80 check fall 3 rise 2
    server worker2 10.240.5.171:80 check fall 3 rise 2

#---------------------------------------------------------------------
# HTTPS Ingress (port 443)
#---------------------------------------------------------------------
frontend ingress_https_frontend
    bind *:443
    mode tcp
    option tcplog
    default_backend ingress_https_backend

backend ingress_https_backend
    mode tcp
    balance roundrobin
    option tcp-check
    option ssl-hello-chk
    # Point to nodes where Ingress Controller runs
    server master1 10.240.5.167:443 check fall 3 rise 2
    server master2 10.240.5.168:443 check fall 3 rise 2
    server master3 10.240.5.169:443 check fall 3 rise 2
    server worker1 10.240.5.172:443 check fall 3 rise 2
    server worker2 10.240.5.171:443 check fall 3 rise 2

    # Or if you have worker nodes:
    # server worker1 10.0.0.21:443 check fall 3 rise 2
    # server worker2 10.0.0.22:443 check fall 3 rise 2


listen stats
    bind *:9000
    stats enable
    stats uri /stats
    stats refresh 30s
```
### 2. Enable and start HAProxy
```bash
systemctl enable haproxy
syatemctl start haproxy
```

### 3. Verify HAProxy status:
```bash
systemctl status haproxy
```
### 4. Install keepalived on both HAProxy node
```bash
For Ubuntu:
apt-get install keepalived

For RHEL:
dnf install keepalived
```
### 5. Create `/etc/keepalived/keepalived.conf` with the following configuration:

```bash
vim /etc/keepalived/keepalived.conf
```
HAProxy #1 – MASTER

Contents of `/etc/keepalived/keepalived.conf` for master:
```bash
vrrp_instance VI_1 {
    state MASTER
    interface ens160            # network interface based on your server
    virtual_router_id 51
    priority 200              # higher = master
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 12345
    }

    virtual_ipaddress {
        10.240.5.164
    }

```

HAProxy #2 – BACKUP

Contents of `/etc/keepalived/keepalived.conf` for backup:
```bash
vrrp_instance VI_1 {
    state BACKUP
    interface ens160            # network interface based on your server
    virtual_router_id 51
    priority 150              # lower = backup
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 12345
    }

    virtual_ipaddress {
        10.240.5.164
    }
}
```
### 6. Enable and start Keepalived on both haproxy server:
```bash
systemctl start keepalived
systemctl enable keepalived
```
Check status of the keepalive on the haproxy master node, it will have two ip on its network interface:
```bash
ip addr
```

# b. RKE2 Installation on Master and Worker Node

(This guide is for Rocky Linux Operating System)

# For Master Node:

### 1. Install RKE2 on master
```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=server sudo sh -
```
### 2. Configure the first master node
Create a file `/etc/rancher/rke2/config.yaml`
```bash
vim /etc/rancher/rke2/config.yaml
```
with the following content:
```bash
token: yourownsecuritytoken
tls-san:
 - ha-proxy-ip
 - server-mastername
 - server-masterip
```
### 3. For kubectl to work for this cluster (ex: kubectl get nodes), you need to copy rke2.yaml file into .kube folder in the `/root` directory.
```bash
cd /root
mkdir .kube
cp /etc/rancher/rke2/rke2.yaml .kube/config
```
### 4. Start RKE2 on the first master node
```bash
systemctl enable rke2-server
systemctl start rke2-server
```
### 5. For other master nodes,follow steps 1 and 2. But the content of the /etc/rancher/rke2/config.yaml is a little bit different. The security token must be the same as the first master node.
Contents of the `etc/rancher/rke2/rke2.yaml` file for worker node:
```bash
server: https://keepalived-ip:9345
token: yourownsecuritytoken
tls-san:
  - keepalived-proxy-ip
  - haproxy-ip
  - haproxy-hostname
```
### 6. Start RKE2 on other master nodes
```bash
systemctl enable rke2-server
systemctl start rke2-serve
```
### 7. List Cluster Nodes to check the status of the cluster
You can use kubectl on the first master nodes since we created .kube there. All the other nodes (including haproxy nodes) can also have this capability which you just need to do the same step 3 for all the nodes. As for the haproxy nodes, you must get the file from one of the master node.

```bash
#list all nodes
kubectl get nodes

#list all pods in all namespace
Kubectl get pods -A
```
### 8. Usually we don't allow the master nodes to run the workload. The command below will taint the master node to not run any app workload.
Execute this command for all master node on a master node that have kubectl tool:
```bash
kubectl taint nodes <node-name> node-role.kubernetes.io/control-plane=:NoSchedule --overwrite
```

# For Worker Nodes:

### 1. Install RKE2 on worker
```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=agent sudo sh -
dnf install rke2-agent
```
### 2. Configure the worker node
Create a file `/etc/rancher/rke2/config.yaml`
```bash
vim /etc/rancher/rke2/config.yaml
```
with the following content:
```bash
server: https://keepalived-ip:9345
token: yourownsecuritytoken
tls-san:
  - keepalived-proxy-ip
  - haproxy-ip
  - haproxy-hostname
```














