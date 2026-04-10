# Installation Guide

## 1. HAProxy & Keepalived Setup
Perform these steps on both HAProxy nodes

### HAProxy Installation
```bash
# For Ubuntu
sudo apt update && sudo apt install haproxy -y
```

## 2. Create `/etc/haproxy/haproxy.cfg` with the following configuration:
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
## 3. Enable and start HAProxy
```bash
systemctl enable haproxy
syatemctl start haproxy
```

## 4. Verify HAProxy status:
```bash
systemctl status haproxy
```
## 5. Install keepalived on both HAProxy node
```bash
For Ubuntu:
apt-get install keepalived

For RHEL:
dnf install keepalived
```
## 6. Create `/etc/keepalived/keepalived.conf` with the following configuration:
```bash


