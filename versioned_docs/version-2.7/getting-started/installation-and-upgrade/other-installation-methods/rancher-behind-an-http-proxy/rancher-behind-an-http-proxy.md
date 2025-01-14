---
title: Installing Rancher behind an HTTP Proxy
---

<head>
  <link rel="canonical" href="https://ranchermanager.docs.rancher.com/pages-for-subheaders/rancher-behind-an-http-proxy"/>
</head>

In a lot of enterprise environments, servers or VMs running on premise do not have direct Internet access, but must connect to external services through a HTTP(S) proxy for security reasons. This tutorial shows step by step how to set up a highly available Rancher installation in such an environment.

Alternatively, it is also possible to set up Rancher completely air-gapped without any Internet access. This process is described in detail in the [Rancher docs](../air-gapped-helm-cli-install/air-gapped-helm-cli-install.md).

## Installation Outline

1. [Set up infrastructure](set-up-infrastructure.md)
2. [Set up a Kubernetes cluster](install-kubernetes.md)
3. [Install Rancher](install-rancher.md)
