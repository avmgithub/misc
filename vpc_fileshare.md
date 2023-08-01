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
Create a VPC File Share. Run the following commmands.

Get the ID for your file share.
```
# ibmcloud is shares
Listing shares in resource group Default and region us-east under account xxx
ID                                          Name         Lifecycle state   Zone        Profile   Size(GB)   Resource group   
r014-a6f3f85e-9658-4a3b-b94e-e65ee9bca3d2   vpcfsroks1   stable            us-east-2   dp2       10         Default   
```

Get the share target.
```
# ibmcloud is share-targets r014-a6f3f85e-9658-4a3b-b94e-e65ee9bca3d2
Listing share target of r014-a6f3f85e-9658-4a3b-b94e-e65ee9bca3d2 in all resource groups and region us-east under account xxx
ID                                          Name        VPC              Lifecycle state   
r014-585adfcd-0a6a-4f75-be8c-32b9c55ebeb8   vpcfsroks   satellite-test   stable   
```

Get the nfsServerPath, also called the Mount Path.
```
ibmcloud is share-target r014-a6f3f85e-9658-4a3b-b94e-e65ee9bca3d2 r014-585adfcd-0a6a-4f75-be8c-32b9c55ebeb8
Getting target ID r014-585adfcd-0a6a-4f75-be8c-32b9c55ebeb8 for share ID r014-a6f3f85e-9658-4a3b-b94e-e65ee9bca3d2 under account xxx
                     
ID                r014-585adfcd-0a6a-4f75-be8c-32b9c55ebeb8   
Name              vpcfsroks   
VPC               ID                                          Name      
                  r014-74e1bfb0-8607-4592-8173-3d08f4fcd704   satellite-test      
                     
Lifecycle state   stable   
Mount path        fsf-wdc0651b-byok-fz.adn.networklayer.com:/f5166891_fb39_4404_a34b_438e63e57a3d   
Created           2023-08-01T07:10:34-05:00   
```

### Step 3

Create a PV configuration file called static-file-share.yaml that references your file share.

Create a PV yaml using this template:
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: static-file-share
spec:
  mountOptions:
    - hard
    - nfsvers=4.1
    - sec=sys
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 10Gi
  csi:
    volumeAttributes:
      nfsServerPath:  NFS-SERVER-PATH
    driver: vpc.file.csi.ibm.io
    volumeHandle: FILE-SHARE-ID:SHARE-TARGET-ID
```

Create a PV like so:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: static-file-share
spec:
  mountOptions:
    - hard
    - nfsvers=4.1
    - sec=sys
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 10Gi
  csi:
    volumeAttributes:
      nfsServerPath:  fsf-wdc0651b-byok-fz.adn.networklayer.com:/f5166891_fb39_4404_a34b_438e63e57a3d
    driver: vpc.file.csi.ibm.io
    volumeHandle: r014-a6f3f85e-9658-4a3b-b94e-e65ee9bca3d2:r014-585adfcd-0a6a-4f75-be8c-32b9c55ebeb8

```

Following was the bad example. Retain just in case. Do not use.
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

```
oc get pv
NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
static-file-share   10Gi       RWX            Retain           Available                                   3s

```


### Step 4
Create a PVC that consume the PV

``` yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-pv
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  volumeName: static-file-share
  storageClassName: '' #Leave the storage class blank.
  volumeMode: Filesystem
```

```
oc get pvc
NAME    STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pv   Bound    static-file-share   10Gi       RWX                           54s
```

```
oc get pv
NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE
static-file-share   10Gi       RWX            Retain           Bound    default/my-pv                           64s
```


Create an example application to use the fileshare
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: nginx
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(date -u) >> /test/test.txt; sleep 5; done"]
    volumeMounts:
    - name: persistent-storage
      mountPath: /test
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: my-pv
```

Create the application
```
# oc create -f app.yaml 
pod/app created
```


**Ensure that `spec.volumeName` matches the name of the PV created**

**Ensure that the spec.storageClassName is ''**


Exec into the pod and see if the fileshare is writtable.
```
oc exec app -it -- bash
root@app:/# cd /test
root@app:/test# ls
test.txt
root@app:/test# tail -f test.txt
Tue Aug 1 13:06:04 UTC 2023
Tue Aug 1 13:06:09 UTC 2023
Tue Aug 1 13:06:14 UTC 2023
Tue Aug 1 13:06:19 UTC 2023

^C
```



At this point, your PVC should show a status of `Bound` in a few seconds. If not, double-check the above YAML definitions
and confirm you gave proper authorization to the NFS volume so your workers can access it. 
