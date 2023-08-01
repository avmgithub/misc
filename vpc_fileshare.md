IBM Cloud IKS and ROKS clusters can leverage NFS-based PVCs out of the box with no additional CSI driver setup. 
This works for VPC-based and classic-based clusters. 

### Step 1
Provision your cluster or skip to step 2 if you already have a cluster up

Login to your IBM cloud account to use the cli.

Run the following command to see if the vpc file share csi driver is visible.

```
# ibmcloud ks cluster addon versions
Name                        Version            Supported Kubernetes Range   Supported OpenShift Range   Kubernetes Default   OpenShift Default
pc-file-csi-driver         1.1 (default)      >=1.21.0 <1.28.0             >=4.7.0 <4.14.0             -                    -
```

Run the command to add the addon to the ROKS cluster
```
# ibmcloud ks cluster addon enable vpc-file-csi-driver --cluster <cluster_id>
Enabling add-on vpc-file-csi-driver for cluster <cluster_id>
The add-on might take several minutes to deploy and become ready for use.
OK
```

It will take a few minutes for the addon to be fully enabled

```
# ibmcloud ks cluster addon ls --cluster <cluster_id>
OK
Name                   Version   Health State   Health Status
vpc-block-csi-driver   5.0       normal         Addon Ready. For more info: http://ibm.biz/addon-state (H1500)
vpc-file-csi-driver    1.1       -              Enabling
```
After a few minutes the addon should be ready.
```
ibmcloud ks cluster addon ls --cluster <cluster_id>
OK
Name                   Version   Health State   Health Status
vpc-block-csi-driver   5.0       normal         Addon Ready. For more info: http://ibm.biz/addon-state (H1500)
vpc-file-csi-driver    1.1       normal         Addon Ready. For more info: http://ibm.biz/addon-state (H1500)
```

### Step 2
Create a VPC File Share or Classic File Volume. Note the NFS server and path. 
> Don't forget to authorize your cluster hosts/subnet to access the NFS share!

### Step 3
Create a PV like so:

``` yaml

kind: PersistentVolume
apiVersion: v1
metadata:
  name: myvol
spec:
  capacity:
    storage: 50Gi
  nfs:
    server: <NFS host, e.g. fsf-wdc0451a-fz.adn.networklayer.com>
    path: <NFS volume path, e.g. /nxg_s_voll_mz0757_3bcce10f_dbb9_4241_b3bb_d77785e68376>
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  storageClassName: ""

```

**Ensure that the `spec.capacity.storage` value matches the actual NFS volume capacity** 

**Ensure that the spec.storageClassName is ""**

### Step 4
Create a PVC that consume the PV

``` yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-pv
  namespace: my-namespace
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Gi
  volumeName: myvol
  storageClassName: ''
  volumeMode: Filesystem
```

**Ensure that `spec.volumeName` matches the name of the PV created**

**Ensure that the spec.storageClassName is ''**

At this point, your PVC should show a status of `Bound` in a few seconds. If not, double-check the above YAML definitions
and confirm you gave proper authorization to the NFS volume so your workers can access it. 
