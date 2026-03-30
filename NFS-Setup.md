# NFS Setup for k8s cluster
This section is for setting up the nfs storage for the k8s cluster. All the steps below are in the right order. Please follow step by step.

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

## Setup StorageClass and test using PVC

1. Create a StorageClassYAML file `nfs-storageclass.yaml` and add all the lines as below:
```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
allowVolumeExpansion: true
parameters:
  server: <YOUR_NFS_SERVER_IP>
  share: </datastore> #Path that you exports on the nfs server
reclaimPolicy: Delete
volumeBindingMode: Immediate
mountOptions:
  - nfsvers=4.1
```
Apply the yaml file to create the storageclass:
```bash
kubcetl apply -f nfs-storageclass.yaml
```

2. Create a PVC using the StorageClass:
Create a `test-pvc.yaml` file, add the followings lines and apply the yaml file to test the pvc.
```bash
#Contents of tapiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-test-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-csi  # This MUST match the 'name' in your StorageClass YAML
  resources:
    requests:
      storage: 1Gi file
```
Apply the yaml file to create the PVC
```bash
kubectl apply -f test-pvc.yaml
```
3. Create a test pod nfs-test-pod.yaml
```bash
#YAML
apiVersion: v1
kind: Pod
metadata:
  name: nfs-installer-test
spec:
  containers:
  - name: shell
    image: busybox
    command:
      - "/bin/sh"
      - "-c"
      - "while true; do echo $(date) - Hello from RKE2 NFS >> /mnt/test-data.txt; sleep 5; done"
    volumeMounts:
    - name: nfs-storage
      mountPath: /mnt
  volumes:
  - name: nfs-storage
    persistentVolumeClaim:
      claimName: nfs-test-pvc # This must match your PVC name
```
Apply the yaml file to create the pod
```bash
kubectl apply -f nfs-test-pod.yaml
```
4. Verify the Write Operation
When the pod is running Check the Pod logs/files:
```bash
kubectl exec nfs-installer-test -- cat /mnt/test-data.txt
```

Go to the nfs server and check the content of file in the path that you export:

```bash
cat /datastore/pvc-xxxxyourfilepv/test-data.txt
```
5. Test the nfs with the developer to confirm the nfs setup works.




