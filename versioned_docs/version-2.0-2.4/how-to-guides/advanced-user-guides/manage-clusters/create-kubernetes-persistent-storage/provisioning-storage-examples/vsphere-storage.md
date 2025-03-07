---
title: vSphere Storage
---

<head>
  <link rel="canonical" href="https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/manage-clusters/provisioning-storage-examples/vsphere-storage"/>
</head>

To provide stateful workloads with vSphere storage, we recommend creating a vSphereVolume StorageClass. This practice dynamically provisions vSphere storage when workloads request volumes through a [persistent volume claim](../../../../../pages-for-subheaders/create-kubernetes-persistent-storage.md).

In order to dynamically provision storage in vSphere, the vSphere provider must be [enabled.](../../../../new-user-guides/kubernetes-clusters-in-rancher-setup/launch-kubernetes-with-rancher/set-up-cloud-providers/other-cloud-providers/vsphere.md)


### Prerequisites

In order to provision vSphere volumes in a cluster created with the [Rancher Kubernetes Engine (RKE)](../../../../../pages-for-subheaders/launch-kubernetes-with-rancher.md), the [vSphere cloud provider](https://rancher.com/docs/rke/latest/en/config-options/cloud-providers/vsphere) must be explicitly enabled in the [cluster options](../../../../../reference-guides/cluster-configuration/rancher-server-configuration/rke1-cluster-configuration.md).

### Creating a StorageClass

> **Note:**
>
> The following steps can also be performed using the `kubectl` command line tool. See [Kubernetes documentation on persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) for details.

1. From the Global view, open the cluster where you want to provide vSphere storage.
2. From the main menu, select **Storage > Storage Classes**. Then click **Add Class**.
3. Enter a **Name** for the class.
4. Under **Provisioner**, select **VMWare vSphere Volume**.

    ![](/img/vsphere-storage-class.png)

5. Optionally, specify additional properties for this storage class under **Parameters**. Refer to the [vSphere storage documentation](https://github.com/vmware-archive/vsphere-storage-for-kubernetes/blob/master/documentation/storageclass.md) for details.
5. Click **Save**.

### Creating a Workload with a vSphere Volume

1. From the cluster where you configured vSphere storage, begin creating a workload as you would in [Deploying Workloads](../../../../new-user-guides/kubernetes-resources-setup/workloads-and-pods/deploy-workloads.md).
2. For **Workload Type**, select **Stateful set of 1 pod**.
3. Expand the **Volumes** section and click **Add Volume**.
4. Choose **Add a new persistent volume (claim)**. This option will implicitly create the claim once you deploy the workload.
5. Assign a **Name** for the claim, ie. `test-volume` and select the vSphere storage class created in the previous step.
6. Enter the required **Capacity** for the volume. Then click **Define**.

    ![](/img/workload-add-volume.png)

7. Assign a path in the **Mount Point** field. This is the full path where the volume will be mounted in the container file system, e.g. `/persistent`.
8. Click **Launch** to create the workload.

### Verifying Persistence of the Volume

1. From the context menu of the workload you just created, click **Execute Shell**.
2. Note the directory at root where the volume has been mounted to (in this case `/persistent`).
3. Create a file in the volume by executing the command `touch /<volumeMountPoint>/data.txt`.
4. **Close** the shell window.
5. Click on the name of the workload to reveal detail information.
6. Open the context menu next to the Pod in the *Running* state.
7. Delete the Pod by selecting **Delete**.
8. Observe that the pod is deleted. Then a new pod is scheduled to replace it so that the workload maintains its configured scale of a single stateful pod.
9. Once the replacement pod is running, click **Execute Shell**.
10. Inspect the contents of the directory where the volume is mounted by entering `ls -l /<volumeMountPoint>`. Note that the file you created earlier is still present.

    ![workload-persistent-data](/img/workload-persistent-data.png)

### Why to Use StatefulSets Instead of Deployments

You should always use [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) for workloads consuming vSphere storage, as this resource type is designed to address a VMDK block storage caveat.

Since vSphere volumes are backed by VMDK block storage, they only support an [access mode](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) of `ReadWriteOnce`. This setting restricts the volume so that it can only be mounted to a single pod at a time, unless all pods consuming that volume are co-located on the same node. This behavior makes a deployment resource unusable for scaling beyond a single replica if it consumes vSphere volumes.

Even using a deployment resource with just a single replica may result in a deadlock situation while updating the deployment. If the updated pod is scheduled to a node different from where the existing pod lives, it will fail to start because the VMDK is still attached to the other node.

### Related Links

- [vSphere Storage for Kubernetes](https://github.com/vmware-archive/vsphere-storage-for-kubernetes/tree/master/documentation)
- [Kubernetes Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
