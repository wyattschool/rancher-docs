---
title: Kubernetes Resources
---

<head>
  <link rel="canonical" href="https://ranchermanager.docs.rancher.com/pages-for-subheaders/kubernetes-resources-setup"/>
</head>

> The <b>Cluster Explorer</b> is a new feature in Rancher v2.5 that allows you to view and manipulate all of the custom resources and CRDs in a Kubernetes cluster from the Rancher UI. This section will be updated to reflect the way that Kubernetes resources are handled in Rancher v2.5.

## Workloads

Deploy applications to your cluster nodes using [workloads](workloads-and-pods/workloads-and-pods.md), which are objects that contain pods that run your apps, along with metadata that set rules for the deployment's behavior. Workloads can be deployed within the scope of the entire clusters or within a namespace.

When deploying a workload, you can deploy from any image. There are a variety of [workload types](workloads-and-pods/workloads-and-pods.md#workload-types) to choose from which determine how your application should run.

Following a workload deployment, you can continue working with it. You can:

- [Upgrade](workloads-and-pods/upgrade-workloads.md) the workload to a newer version of the application it's running.
- [Roll back](workloads-and-pods/roll-back-workloads.md) a workload to a previous version, if an issue occurs during upgrade.
- [Add a sidecar](workloads-and-pods/add-a-sidecar.md), which is a workload that supports a primary workload.

## Load Balancing and Ingress

### Load Balancers

After you launch an application, it's only available within the cluster. It can't be reached externally.

If you want your applications to be externally accessible, you must add a load balancer to your cluster. Load balancers create a gateway for external connections to access your cluster, provided that the user knows the load balancer's IP address and the application's port number.

Rancher supports two types of load balancers:

- [Layer-4 Load Balancers](load-balancer-and-ingress-controller/layer-4-and-layer-7-load-balancing.md#layer-4-load-balancer)
- [Layer-7 Load Balancers](load-balancer-and-ingress-controller/layer-4-and-layer-7-load-balancing.md#layer-7-load-balancer)

For more information, see [load balancers](load-balancer-and-ingress-controller/layer-4-and-layer-7-load-balancing.md).

#### Ingress

Load Balancers can only handle one IP address per service, which means if you run multiple services in your cluster, you must have a load balancer for each service. Running multiples load balancers can be expensive. You can get around this issue by using an ingress.

Ingress is a set of rules that act as a load balancer. Ingress works in conjunction with one or more ingress controllers to dynamically route service requests. When the ingress receives a request, the ingress controller(s) in your cluster program the load balancer to direct the request to the correct service based on service subdomains or path rules that you've configured.

For more information, see [Ingress](load-balancer-and-ingress-controller/add-ingresses.md).

When using ingresses in a project, you can program the ingress hostname to an external DNS by setting up a Global DNS entry.

## Service Discovery

After you expose your cluster to external requests using a load balancer and/or ingress, it's only available by IP address. To create a resolveable hostname, you must create a service record, which is a record that maps an IP address, external hostname, DNS record alias, workload(s), or labelled pods to a specific hostname.

For more information, see [Service Discovery](create-services.md).


## Applications

Besides launching individual components of an application, you can use the Rancher catalog to start launching applications, which are Helm charts.

For more information, see [Applications in a Project](../helm-charts-in-rancher.md).

## Kubernetes Resources

Within the context of a Rancher project or namespace, _resources_ are files and data that support operation of your pods. Within Rancher, certificates, registries, and secrets are all considered resources. However, Kubernetes classifies resources as different types of [secrets](https://kubernetes.io/docs/concepts/configuration/secret/). Therefore, within a single project or namespace, individual resources must have unique names to avoid conflicts. Although resources are primarily used to carry sensitive information, they have other uses as well.

Resources include:

- [Certificates](encrypt-http-communication.md): Files used to encrypt/decrypt data entering or leaving the cluster.
- [ConfigMaps](configmaps.md): Files that store general configuration information, such as a group of config files.
- [Secrets](secrets.md): Files that store sensitive data like passwords, tokens, or keys.
- [Registries](kubernetes-and-docker-registries.md): Files that carry credentials used to authenticate with private registries.
