---
title: Enabling Experimental Features
---

<head>
  <link rel="canonical" href="https://ranchermanager.docs.rancher.com/pages-for-subheaders/enable-experimental-features"/>
</head>

Rancher includes some features that are experimental and disabled by default. You might want to enable these features, for example, if you decide that the benefits of using an [unsupported storage type](../getting-started/installation-and-upgrade/advanced-options/enable-experimental-features/unsupported-storage-drivers.md) outweighs the risk of using an untested feature. Feature flags were introduced to allow you to try these features that are not enabled by default.

The features can be enabled in three ways:

- [Enable features when starting Rancher.](#enabling-features-when-starting-rancher) When installing Rancher with a CLI, you can use a feature flag to enable a feature by default.
- [Enable features from the Rancher UI](#enabling-features-with-the-rancher-ui) in Rancher v2.3.3+ by going to the **Settings** page.
- [Enable features with the Rancher API](#enabling-features-with-the-rancher-api) after installing Rancher.

Each feature has two values:

- A default value, which can be configured with a flag or environment variable from the command line
- A set value, which can be configured with the Rancher API or UI

If no value has been set, Rancher uses the default value.

Because the API sets the actual value and the command line sets the default value, that means that if you enable or disable a feature with the API or UI, it will override any value set with the command line.

For example, if you install Rancher, then set a feature flag to true with the Rancher API, then upgrade Rancher with a command that sets the feature flag to false, the default value will still be false, but the feature will still be enabled because it was set with the Rancher API. If you then deleted the set value (true) with the Rancher API, setting it to NULL, the default value (false) would take effect. See the [feature flags page](../reference-guides/installation-references/feature-flags.md) for more information.

## Enabling Features when Starting Rancher

When you install Rancher, enable the feature you want with a feature flag. The command is different depending on whether you are installing Rancher on a single node or if you are doing a Kubernetes Installation of Rancher.

> **Note:** Values set from the Rancher API will override the value passed in through the command line.

<Tabs>
<TabItem value="Kubernetes Install">

When installing Rancher with a Helm chart, use the `--features` option. In the below example, two features are enabled by passing the feature flag names names in a comma separated list:

```
helm install rancher-latest/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set 'extraEnv[0].name=CATTLE_FEATURES' # Available as of v2.3.0
  --set 'extraEnv[0].value=<FEATURE-FLAG-NAME-1>=true,<FEATURE-FLAG-NAME-2>=true' # Available as of v2.3.0
```

Note: If you are installing an alpha version, Helm requires adding the `--devel` option to the command.

### Rendering the Helm Chart for Air Gap Installations

For an air gap installation of Rancher, you need to add a Helm chart repository and render a Helm template before installing Rancher with Helm. For details, refer to the [air gap installation documentation.](../getting-started/installation-and-upgrade/other-installation-methods/air-gapped-helm-cli-install/install-rancher-ha.md)

Here is an example of a command for passing in the feature flag names when rendering the Helm template. In the below example, two features are enabled by passing the feature flag names in a comma separated list.

The Helm 3 command is as follows:

```
helm template rancher ./rancher-<VERSION>.tgz --output-dir . \
  --namespace cattle-system \
  --set hostname=<RANCHER.YOURDOMAIN.COM> \
  --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher \
  --set ingress.tls.source=secret \
  --set systemDefaultRegistry=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Available as of v2.2.0, set a default private registry to be used in Rancher
  --set useBundledSystemChart=true # Available as of v2.3.0, use the packaged Rancher system charts
  --set 'extraEnv[0].name=CATTLE_FEATURES' # Available as of v2.3.0
  --set 'extraEnv[0].value=<FEATURE-FLAG-NAME-1>=true,<FEATURE-FLAG-NAME-2>=true' # Available as of v2.3.0
```

The Helm 2 command is as follows:

```
helm template ./rancher-<VERSION>.tgz --output-dir . \
  --name rancher \
  --namespace cattle-system \
  --set hostname=<RANCHER.YOURDOMAIN.COM> \
  --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher \
  --set ingress.tls.source=secret \
  --set systemDefaultRegistry=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Available as of v2.2.0, set a default private registry to be used in Rancher
  --set useBundledSystemChart=true # Available as of v2.3.0, use the packaged Rancher system charts
  --set 'extraEnv[0].name=CATTLE_FEATURES' # Available as of v2.3.0
  --set 'extraEnv[0].value=<FEATURE-FLAG-NAME-1>=true,<FEATURE-FLAG-NAME-2>=true' # Available as of v2.3.0
```


</TabItem>
<TabItem value="Docker Install">

When installing Rancher with Docker, use the `--features` option. In the below example, two features are enabled by passing the feature flag names in a comma separated list:

```
docker run -d -p 80:80 -p 443:443 \
  --restart=unless-stopped \
  rancher/rancher:rancher-latest \
  --features=<FEATURE-FLAG-NAME-1>=true,<FEATURE-FLAG-NAME-2>=true # Available as of v2.3.0
```

</TabItem>
</Tabs>

## Enabling Features with the Rancher UI

1. Go to the **Global** view and click **Settings.**
1. Click the **Feature Flags** tab. You will see a list of experimental features.
1. To enable a feature, go to the disabled feature you want to enable and click **&#8942; > Activate.**

**Result:** The feature is enabled.

### Disabling Features with the Rancher UI

1. Go to the **Global** view and click **Settings.**
1. Click the **Feature Flags** tab. You will see a list of experimental features.
1. To disable a feature, go to the enabled feature you want to disable and click **&#8942; > Deactivate.**

**Result:** The feature is disabled.

## Enabling Features with the Rancher API

1. Go to `<RANCHER-SERVER-URL>/v3/features`.
1. In the `data` section, you will see an array containing all of the features that can be turned on with feature flags. The name of the feature is in the `id` field. Click the name of the feature you want to enable.
1. In the upper left corner of the screen, under **Operations,** click **Edit.**
1. In the **Value** drop-down menu, click **True.**
1. Click **Show Request.**
1. Click **Send Request.**
1. Click **Close.**

**Result:** The feature is enabled.

### Disabling Features with the Rancher API

1. Go to `<RANCHER-SERVER-URL>/v3/features`.
1. In the `data` section, you will see an array containing all of the features that can be turned on with feature flags. The name of the feature is in the `id` field. Click the name of the feature you want to enable.
1. In the upper left corner of the screen, under **Operations,** click **Edit.**
1. In the **Value** drop-down menu, click **False.**
1. Click **Show Request.**
1. Click **Send Request.**
1. Click **Close.**

**Result:** The feature is disabled.
