---
title: Install/Upgrade Rancher on a Kubernetes Cluster
description: Learn how to install Rancher in development and production environments. Read about single node and high availability installation
---

<head>
  <link rel="canonical" href="https://ranchermanager.docs.rancher.com/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster"/>
</head>

In this section, you'll learn how to deploy Rancher on a Kubernetes cluster using the Helm CLI.

## Prerequisites

- [Kubernetes Cluster](#kubernetes-cluster)
- [CLI Tools](#cli-tools)
- [Ingress Controller (Only for Hosted Kubernetes)](#ingress-controller-for-hosted-kubernetes)

### Kubernetes Cluster

Set up the Rancher server's local Kubernetes cluster.

Rancher can be installed on any Kubernetes cluster. This cluster can use upstream Kubernetes, or it can use one of Rancher's Kubernetes distributions, or it can be a managed Kubernetes cluster from a provider such as Amazon EKS.

For help setting up a Kubernetes cluster, we provide these tutorials:

- **RKE:** For the tutorial to install an RKE Kubernetes cluster, refer to [this page.](../../../how-to-guides/new-user-guides/kubernetes-cluster-setup/rke1-for-rancher.md) For help setting up the infrastructure for a high-availability RKE cluster, refer to [this page.](../../../how-to-guides/new-user-guides/infrastructure-setup/ha-rke1-kubernetes-cluster.md)
- **K3s:** For the tutorial to install a K3s Kubernetes cluster, refer to [this page.](../../../how-to-guides/new-user-guides/kubernetes-cluster-setup/k3s-for-rancher.md) For help setting up the infrastructure for a high-availability K3s cluster, refer to [this page.](../../../how-to-guides/new-user-guides/infrastructure-setup/ha-k3s-kubernetes-cluster.md)
- **RKE2:** For the tutorial to install an RKE2 Kubernetes cluster, refer to [this page.](../../../how-to-guides/new-user-guides/kubernetes-cluster-setup/rke2-for-rancher.md) For help setting up the infrastructure for a high-availability RKE2 cluster, refer to [this page.](../../../how-to-guides/new-user-guides/infrastructure-setup/ha-rke2-kubernetes-cluster.md)
- **Amazon EKS:** For details on how to install Rancher on Amazon EKS, including how to install an ingress so that the Rancher server can be accessed, refer to [this page.](rancher-on-amazon-eks.md)
- **AKS:** For details on how to install Rancher with Azure Kubernetes Service, including how to install an ingress so that the Rancher server can be accessed, refer to [this page.](rancher-on-aks.md)
- **GKE:** For details on how to install Rancher with Google Kubernetes Engine, including how to install an ingress so that the Rancher server can be accessed, refer to [this page.](rancher-on-gke.md)

### CLI Tools

The following CLI tools are required for setting up the Kubernetes cluster. Please make sure these tools are installed and available in your `$PATH`.

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl) - Kubernetes command-line tool.
- [helm](https://docs.helm.sh/using_helm/#installing-helm) - Package management for Kubernetes. Refer to the [Helm version requirements](../resources/helm-version-requirements.md) to choose a version of Helm to install Rancher. Refer to the [instructions provided by the Helm project](https://helm.sh/docs/intro/install/) for your specific platform.

### Ingress Controller (For Hosted Kubernetes)

To deploy Rancher v2.5 on a hosted Kubernetes cluster such as EKS, GKE, or AKS, you should deploy a compatible Ingress controller first to configure [SSL termination on Rancher.](#3-choose-your-ssl-configuration)

For an example of how to deploy an ingress on EKS, refer to [this section.](rancher-on-amazon-eks.md#5-install-an-ingress)

## Install the Rancher Helm Chart

Rancher is installed using the Helm package manager for Kubernetes. Helm charts provide templating syntax for Kubernetes YAML manifest documents.

With Helm, we can create configurable deployments instead of just using static files. For more information about creating your own catalog of deployments, check out the docs at https://helm.sh/.

For systems without direct internet access, see [Air Gap: Kubernetes install](../other-installation-methods/air-gapped-helm-cli-install/install-rancher-ha.md).

To choose a Rancher version to install, refer to [Choosing a Rancher Version.](../resources/choose-a-rancher-version.md)

To choose a version of Helm to install Rancher with, refer to the [Helm version requirements](../resources/helm-version-requirements.md)

> **Note:** The installation instructions assume you are using Helm 3. For migration of installs started with Helm 2, refer to the official [Helm 2 to 3 migration docs.](https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/) This [section](../resources/helm-version-requirements.md) provides a copy of the older installation instructions for Rancher installed on an RKE Kubernetes cluster with Helm 2, and it is intended to be used if upgrading to Helm 3 is not feasible.

To set up Rancher,

1. [Add the Helm chart repository](#1-add-the-helm-chart-repository)
2. [Create a namespace for Rancher](#2-create-a-namespace-for-rancher)
3. [Choose your SSL configuration](#3-choose-your-ssl-configuration)
4. [Install cert-manager](#4-install-cert-manager) (unless you are bringing your own certificates, or TLS will be terminated on a load balancer)
5. [Install Rancher with Helm and your chosen certificate option](#5-install-rancher-with-helm-and-your-chosen-certificate-option)
6. [Verify that the Rancher server is successfully deployed](#6-verify-that-the-rancher-server-is-successfully-deployed)
7. [Save your options](#7-save-your-options)

### 1. Add the Helm Chart Repository

Use `helm repo add` command to add the Helm chart repository that contains charts to install Rancher. For more information about the repository choices and which is best for your use case, see [Choosing a Rancher Version](../resources/choose-a-rancher-version.md).

- Latest: Recommended for trying out the newest features
    ```
    helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
    ```
- Stable: Recommended for production environments
    ```
    helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
    ```
- Alpha: Experimental preview of upcoming releases.
    ```
    helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
    ```
    Note: Upgrades are not supported to, from, or between Alphas.

### 2. Create a Namespace for Rancher

We'll need to define a Kubernetes namespace where the resources created by the Chart should be installed. This should always be `cattle-system`:

```
kubectl create namespace cattle-system
```

### 3. Choose your SSL Configuration

The Rancher management server is designed to be secure by default and requires SSL/TLS configuration.

> **Note:** If you want terminate SSL/TLS externally, see [TLS termination on an External Load Balancer](../../../reference-guides/installation-references/helm-chart-options.md#external-tls-termination).

There are three recommended options for the source of the certificate used for TLS termination at the Rancher server:

- **Rancher-generated TLS certificate:** In this case, you will need to install `cert-manager` into the cluster. Rancher utilizes `cert-manager` to issue and maintain its certificates. Rancher will generate a CA certificate of its own, and sign a cert using that CA. `cert-manager` is then responsible for managing that certificate.
- **Let's Encrypt:** The Let's Encrypt option also uses `cert-manager`. However, in this case, cert-manager is combined with a special Issuer for Let's Encrypt that performs all actions (including request and validation) necessary for getting a Let's Encrypt issued cert. This configuration uses HTTP validation (`HTTP-01`), so the load balancer must have a public DNS record and be accessible from the internet.
- **Bring your own certificate:** This option allows you to bring your own public- or private-CA signed certificate. Rancher will use that certificate to secure websocket and HTTPS traffic. In this case, you must upload this certificate (and associated key) as PEM-encoded files with the name `tls.crt` and `tls.key`. If you are using a private CA, you must also upload that certificate. This is due to the fact that this private CA may not be trusted by your nodes. Rancher will take that CA certificate, and generate a checksum from it, which the various Rancher components will use to validate their connection to Rancher.


| Configuration                  | Helm Chart Option           | Requires cert-manager                 |
| ------------------------------ | ----------------------- | ------------------------------------- |
| Rancher Generated Certificates (Default) | `ingress.tls.source=rancher`  | [yes](#4-install-cert-manager) |
| Let’s Encrypt                  | `ingress.tls.source=letsEncrypt`  | [yes](#4-install-cert-manager) |
| Certificates from Files        | `ingress.tls.source=secret`        | no               |

### 4. Install cert-manager

> You should skip this step if you are bringing your own certificate files (option `ingress.tls.source=secret`), or if you use [TLS termination on an external load balancer](../../../reference-guides/installation-references/helm-chart-options.md#external-tls-termination).

This step is only required to use certificates issued by Rancher's generated CA (`ingress.tls.source=rancher`) or to request Let's Encrypt issued certificates (`ingress.tls.source=letsEncrypt`).

<details id="cert-manager">
  <summary>Click to Expand</summary>

> **Important:** Recent changes to cert-manager require an upgrade. If you are upgrading Rancher and using a version of cert-manager older than v0.11.0, please see our [upgrade documentation](../resources/upgrade-cert-manager.md/).

These instructions are adapted from the [official cert-manager documentation](https://cert-manager.io/docs/installation/kubernetes/#installing-with-helm).

```
# If you have installed the CRDs manually instead of with the `--set installCRDs=true` option added to your Helm install command, you should upgrade your CRD resources before upgrading the Helm chart:
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.1/cert-manager.crds.yaml

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.1
```

Once you’ve installed cert-manager, you can verify it is deployed correctly by checking the cert-manager namespace for running pods:

```
kubectl get pods --namespace cert-manager

NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-5c6866597-zw7kh               1/1     Running   0          2m
cert-manager-cainjector-577f6d9fd7-tr77l   1/1     Running   0          2m
cert-manager-webhook-787858fcdb-nlzsq      1/1     Running   0          2m
```

</details>

### 5. Install Rancher with Helm and Your Chosen Certificate Option

The exact command to install Rancher differs depending on the certificate configuration.

However, irrespective of the certificate configuration, the name of the Rancher installation in the `cattle-system` namespace should always be `rancher`.

<Tabs>
<TabItem value="Rancher-generated Certificates">

The default is for Rancher to generate a self-signed CA, and uses `cert-manager` to issue the certificate for access to the Rancher server interface.

Because `rancher` is the default option for `ingress.tls.source`, we are not specifying `ingress.tls.source` when running the `helm install` command.

- Set `hostname` to the DNS record that resolves to your load balancer.
- Set `replicas` to the number of replicas to use for the Rancher Deployment. This defaults to 3; if you have less than 3 nodes in your cluster you should reduce it accordingly.
- To install a specific Rancher version, use the `--version` flag, example: `--version 2.3.6`.
- If you are installing an alpha version, Helm requires adding the `--devel` option to the command.

```
helm install rancher rancher-<CHART_REPO>/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set replicas=3
```

Wait for Rancher to be rolled out:

```
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

</TabItem>
<TabItem value="Let's Encrypt">

This option uses `cert-manager` to automatically request and renew [Let's Encrypt](https://letsencrypt.org/) certificates. This is a free service that provides you with a valid certificate as Let's Encrypt is a trusted CA.

>**Note:** You need to have port 80 open as the HTTP-01 challenge can only be done on port 80.

In the following command,

- Set `hostname` to the public DNS record that resolves to your load balancer.
- Set `replicas` to the number of replicas to use for the Rancher Deployment. This defaults to 3; if you have less than 3 nodes in your cluster you should reduce it accordingly.
- Set `ingress.tls.source` to `letsEncrypt`.
- Set `letsEncrypt.email` to the email address used for communication about your certificate (for example, expiry notices).
- Set `letsEncrypt.ingress.class` to whatever your ingress controller is, e.g., `traefik`, `nginx`, `haproxy`, etc.
- To install a specific Rancher version, use the `--version` flag, example: `--version 2.3.6`.
- If you are installing an alpha version, Helm requires adding the `--devel` option to the command.

```
helm install rancher rancher-<CHART_REPO>/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set replicas=3 \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=me@example.org \
  --set letsEncrypt.ingress.class=nginx
```

Wait for Rancher to be rolled out:

```
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

</TabItem>
<TabItem value="Certificates from Files">
In this option, Kubernetes secrets are created from your own certificates for Rancher to use.

When you run this command, the `hostname` option must match the `Common Name` or a `Subject Alternative Names` entry in the server certificate, or the Ingress controller will fail to configure correctly.

Although an entry in the `Subject Alternative Names` is technically required, having a matching `Common Name` maximizes compatibility with older browsers and applications.

> If you want to check if your certificates are correct, see [How do I check Common Name and Subject Alternative Names in my server certificate?](../../../faq/technical-items.md#how-do-i-check-common-name-and-subject-alternative-names-in-my-server-certificate)

- Set `hostname` as appropriate for your certificate, as described above.
- Set `replicas` to the number of replicas to use for the Rancher Deployment. This defaults to 3; if you have less than 3 nodes in your cluster you should reduce it accordingly.
- Set `ingress.tls.source` to `secret`.
- To install a specific Rancher version, use the `--version` flag, example: `--version 2.3.6`.
- If you are installing an alpha version, Helm requires adding the `--devel` option to the command.

```
helm install rancher rancher-<CHART_REPO>/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set replicas=3 \
  --set ingress.tls.source=secret
```

If you are using a Private CA signed certificate , add `--set privateCA=true` to the command:

```
helm install rancher rancher-<CHART_REPO>/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=secret \
  --set privateCA=true
```

Now that Rancher is deployed, see [Adding TLS Secrets](../resources/add-tls-secrets.md) to publish the certificate files so Rancher and the Ingress controller can use them.

</TabItem>
</Tabs>

The Rancher chart configuration has many options for customizing the installation to suit your specific environment. Here are some common advanced scenarios.

- [HTTP Proxy](../../../reference-guides/installation-references/helm-chart-options.md#http-proxy)
- [Private Docker Image Registry](../../../reference-guides/installation-references/helm-chart-options.md#private-registry-and-air-gap-installs)
- [TLS Termination on an External Load Balancer](../../../reference-guides/installation-references/helm-chart-options.md#external-tls-termination)

See the [Chart Options](../../../reference-guides/installation-references/helm-chart-options.md) for the full list of options.


### 6. Verify that the Rancher Server is Successfully Deployed

After adding the secrets, check if Rancher was rolled out successfully:

```
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

If you see the following error: `error: deployment "rancher" exceeded its progress deadline`, you can check the status of the deployment by running the following command:

```
kubectl -n cattle-system get deploy rancher
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rancher   3         3         3            3           3m
```

It should show the same count for `DESIRED` and `AVAILABLE`.

### 7. Save Your Options

Make sure you save the `--set` options you used. You will need to use the same options when you upgrade Rancher to new versions with Helm.

### Finishing Up

That's it. You should have a functional Rancher server.

In a web browser, go to the DNS name that forwards traffic to your load balancer. Then you should be greeted by the colorful login page.

Doesn't work? Take a look at the [Troubleshooting](troubleshooting.md) Page


### Optional Next Steps

Enable the Enterprise Cluster Manager.
