# Backup and Restore for Kubernetes Cluster

## For Etcd backup (Backup dump like):

### Backup Etcd Snapshot

RKE2 backups up the cluster information using etcd snapshots. The snapshots can be scheduled or manually done. 
The snapshots are stored on the node file system (path: `/var/lib/rancher/rke2/server/db/snapshots/` ).

To list the snapshots:
#command only can be run on the master node in the cluster
```bash

rke2 etcd-snapshot list

```
Scheduled snapshots are enabled by default, at 00:00 and 12:00 system time, with 5 snapshots retained. 
Scheduled snapshots have a name that starts with etcd-snapshot, followed by the node name and timestamp.

This is how to manually create snapshots:
```bash
rke2 etcd-snapshot save --name /var/lib/rancher/rke2/server/db/snapshots/manual-snapshot.db
```

### Restore Etcd Snapshot
#### 1. Login to one of the master node
#### 2. Stop rke2-server service
```bash
systemctl stop rke2-server
```
#### 3. Restore using of the snapshot in `/var/lib/rancher/rke2/server/db/snapshots/`:
```bash
sudo rke2 server --cluster-reset --etcd-s3=false --cluster-reset-restore-path=/var/lib/rancher/rke2/server/db/snapshots/manual-snapshot.db
```
#### 4. Start rke2-server service
```bash
systemctl start rke2-server
```
#### 5. Verify cluster health
```bash
kubectl get nodes
```
