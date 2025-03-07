---
title: Setting up Cloud Providers
---

<head>
  <link rel="canonical" href="https://ranchermanager.docs.rancher.com/pages-for-subheaders/set-up-cloud-providers"/>
</head>

A _cloud provider_ is a module in Kubernetes that provides an interface for managing nodes, load balancers, and networking routes. For more information, refer to the [official Kubernetes documentation on cloud providers.](https://github.com/kubernetes/website/blob/release-1.18/content/en/docs/concepts/cluster-administration/cloud-providers.md)

When a cloud provider is set up in Rancher, the Rancher server can automatically provision new nodes, load balancers or persistent storage devices when launching Kubernetes definitions, if the cloud provider you're using supports such automation.

Your cluster will not provision correctly if you configure a cloud provider cluster of nodes that do not meet the prerequisites.

By default, the **Cloud Provider** option is set to `None`.

The following cloud providers can be enabled:

* Amazon
* Azure
* GCE (Google Compute Engine)
* vSphere

### Setting up the Amazon Cloud Provider

For details on enabling the Amazon cloud provider, refer to [this page.](../how-to-guides/new-user-guides/kubernetes-clusters-in-rancher-setup/launch-kubernetes-with-rancher/set-up-cloud-providers/other-cloud-providers/amazon.md)

### Setting up the Azure Cloud Provider

For details on enabling the Azure cloud provider, refer to [this page.](../how-to-guides/new-user-guides/kubernetes-clusters-in-rancher-setup/launch-kubernetes-with-rancher/set-up-cloud-providers/other-cloud-providers/azure.md)

### Setting up the GCE Cloud Provider

For details on enabling the Google Compute Engine cloud provider, refer to [this page.](../how-to-guides/new-user-guides/kubernetes-clusters-in-rancher-setup/launch-kubernetes-with-rancher/set-up-cloud-providers/other-cloud-providers/google-compute-engine.md)

### Setting up the vSphere Cloud Provider

For details on enabling the vSphere cloud provider, refer to [this page.](../how-to-guides/new-user-guides/kubernetes-clusters-in-rancher-setup/launch-kubernetes-with-rancher/set-up-cloud-providers/other-cloud-providers/vsphere.md)

### Setting up a Custom Cloud Provider

The `Custom` cloud provider is available if you want to configure any [Kubernetes cloud provider](https://github.com/kubernetes/website/blob/release-1.18/content/en/docs/concepts/cluster-administration/cloud-providers.md).

For the custom cloud provider option, you can refer to the [RKE docs](https://rancher.com/docs/rke/latest/en/config-options/cloud-providers/) on how to edit the yaml file for your specific cloud provider. There are specific cloud providers that have more detailed configuration :

* [vSphere](https://rke.docs.rancher.com/config-options/cloud-providers/vsphere)
* [OpenStack](https://rancher.com/docs/rke/latest/en/config-options/cloud-providers/openstack/)
