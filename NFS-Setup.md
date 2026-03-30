## Installing NFS CSI driver on Master Node

1. Browse to https://github.com/kubernetes-csi/csi-driver-nfs/tree/master/charts
2. Install CSI driver using helm:
```bash
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --version 4.12.0
```



