## Installing NFS CSI driver on Master Node

1. Browse to https://github.com/kubernetes-csi/csi-driver-nfs/tree/master/charts
2. Install CSI driver using helm:
```bash
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --version 4.12.0
```

## Nfs Server Setup

1. Install nfs on the server:
```bash
dnf install nfs-utils -y
```
2. Prepare the nfs server:

A. Create the directory that you want to export:
```bash
mkdir /datastore
```

B. Set permissions for that directory:
```bash
chmod 755 /datastore
```
C. Configure the Export File:
Edit `/etc/exports` and add the following line to allow rke2 nodes to communciate with it:
```bash
vim /etc/exports
```

Contents ot the file:
```bash
/datastore  10.240.12.0/23(rw,sync,no_subtree_check,no_root_squash)
```

D. Apply the changes:
```bash
exportfs -ra
systemctl start nfs-server
systemctl enable nfs-server
```



