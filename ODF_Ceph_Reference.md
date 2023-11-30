## ODF Ceph references

https://access.redhat.com/articles/4628891
----
[link](https://access.redhat.com/articles/7004852)
----


1) Apply SELinux relabeling workaround for PV/PVC:
https://access.redhat.com/solutions/6906261
for the known OCP issue:
https://access.redhat.com/solutions/6221251
Concern here is that these steps require delete PVC, which we think might be dangerous to perform on Production environment due to risk of data loss.

I. Changing storage device class
-- Modifying the device class where OSD devices are not currently defined with correct class, in this case device class is set to HDD but the backing storage is high tier flash disk (SSD).
This was discussed on the call and customer confirmed the disks are flash media. Below are the steps to make the changes machineconfig and rotational change.
-- Create a new mc group for storage/infra nodes using the article [1] if one does not exist
Custom Machine Config Pool in OpenShift 4

[1]: [link](https://access.redhat.com/solutions/5688941)

[link](https://access.redhat.com/solutions/5688941)

-- Rollout a new mc for udev rules (all of the back-end storage is flash) so rotational = 0
Override device rotational flag in OCS/ODF environments
[2]: https://access.redhat.com/articles/6547891
Once the machine config for udev rules is completed (rotational flag set to 0 for all OSD backing devices), we can proceed with the crushmap changes in ceph
Update the crushmap to reflect the SSD devices:
$ oc rsh rook-ceph-tools-<string>
$ ceph osd tree
For each OSD, remove it from the device class and add it back as SSD. Do 1 OSD at a time, executing both commands for all 60 OSDs.
$ ceph osd crush rm-device-class osd.<id>
$ ceph osd crush set-device-class ssd osd.<id>
II. Selinux relabeling resulting in mount timeouts
-- This was not observed as an issue yet but the file count for file-api-claim is over 618,000 which is quickly approaching the file count we start to see mount timeouts. One of the work-around methods will need to be implemented to mitigate this issue surfacing in the future which is covered in the following article in detail [3]. Not this only applies to cephfs pvc, there 3 or 4 different ways to implement the change but doing this at the storage class level will ensure any cephfs backed pv will have this change alternatively setting on individual PV(s).
Pods using Persistent Volumes with high file counts fail to start or take an excessive amount of time in OpenShift
[3]: https://access.redhat.com/solutions/6221251
III. CPU resources on infra/storage nodes

-- The increase from 8Gi (default) to 16Gi may need to be increased up-to 32Gi as we have seen in other customers with similar workloads, the resources above are from earlier capture (oc describe node <node name>) before the changes, with the increase in memory I believe the node running MDS components was around 83%.
ODF 4.x - How to change CPU and Memory resources on rook-ceph pods
[4]: https://access.redhat.com/solutions/6959127
