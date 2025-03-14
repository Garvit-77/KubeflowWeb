+++
title = "Installation"
description = "How to set up Model Registry"
weight = 30
+++

This section details how to set up and configure Model Registry on your Kubernetes cluster with Kubeflow.

## Prerequisites

These are the minimal requirements to install Model Registry:

- Kubernetes >= 1.27
- Kustomize >= 5.0.3 ([see more](https://github.com/kubeflow/manifests/issues/2388))

<a id="model-registry-install"></a>

## Installing Model Registry

Kubeflow Model registry may be installed as part of a Kubeflow Platform, or as a standalone component.
The best option for you will depend on your specific requirements.

### Installing on Kubeflow Platform

Kubeflow Model Registry is available as an opt-in alpha component in Kubeflow Platform 1.9+, see [Installing Kubeflow](/docs/started/installing-kubeflow/) to learn more about deploying the Kubeflow Platform.

> **Note:** If you are planning to use the Kubeflow UI, please see the section below on installing Model Registry on a Kubeflow Profile. This will ensure proper integration with the Kubeflow Platform and Dashboard.

If you have deployed the Kubeflow manifests, you may follow [these instructions](https://github.com/kubeflow/manifests/tree/master/apps/model-registry/upstream#readme) to deploy Model Registry; please raise any feedback on [`kubeflow/model-registry`](https://github.com/kubeflow/model-registry/issues).

To deploy Model Registry UI, you just need to follow the [UI section](https://github.com/kubeflow/manifests/tree/master/apps/model-registry/upstream#ui-installation) under the Model Registry installation guide targeting a integrated install.

#### Installing on a Kubeflow Profile

Kubeflow Central Dashboard uses [Profiles](/docs/components/central-dash/profiles/) to handle user namespaces and permissions. By default, the manifests deploy the Model Registry in the `kubeflow` namespace, to install a compatible version of Model Registry for Kubeflow, you should first head into the [istio overlay](https://github.com/kubeflow/manifests/tree/master/apps/model-registry/upstream/options/istio) and run the following commands:

```shell
kustomize set namespace <your-profile>
kubectl apply -k .
```

Then head into the [db overlay](https://github.com/kubeflow/manifests/tree/master/apps/model-registry/upstream/overlays/db) and run the following commands:

```shell
kustomize set namespace <your-profile>
kubectl apply -k .
```

### Standalone installation

If you want to install Model Registry separately from Kubeflow, or to get a later version
of Model Registry, you can install the Model Registry manifests directly from the [Model Registry repository](https://github.com/kubeflow/model-registry).

By default, the manifests deploy the Model Registry in the `kubeflow` namespace;
you must ensure the `kubeflow` namespace is available (for example: `kubectl create namespace kubeflow`)
or modify [the kustomization file](https://github.com/kubeflow/model-registry/blob/v{{% model-registry/latest-version %}}/manifests/kustomize/overlays/db/kustomization.yaml#L3) to your desired namespace.

See the list of available versions on the [GitHub releases](https://github.com/kubeflow/model-registry/releases) of the `kubeflow/model-registry` repository. To install a specific release of the Model Registry, modify the following commands with the desired `ref=<GIT_TAG>`.

Run the following command to install the `v{{% model-registry/latest-version %}}` release of Model Registry:

```shell
MODEL_REGISTRY_VERSION={{% model-registry/latest-version %}}
kubectl apply -k "https://github.com/kubeflow/model-registry/manifests/kustomize/overlays/db?ref=v${MODEL_REGISTRY_VERSION}"
```

If your Kubernetes cluster uses Istio, you MUST apply the Istio-compatibility manifests (e.g. when using a full Kubeflow Platform). However, these are NOT required for non-Istio clusters.

```shell
MODEL_REGISTRY_VERSION={{% model-registry/latest-version %}}
kubectl apply -k "https://github.com/kubeflow/model-registry/manifests/kustomize/options/istio?ref=v${MODEL_REGISTRY_VERSION}"
```

If you want Kserve to be able to support `model-registry://` URI formats, you must apply the cluster-scoped `CustomStorageContainer` CR.

```shell
MODEL_REGISTRY_VERSION={{% model-registry/latest-version %}}
kubectl apply -k "https://github.com/kubeflow/model-registry/manifests/kustomize/options/csi?ref=v${MODEL_REGISTRY_VERSION}"
```

## Check Model Registry setup

You can check the status of the Model Registry deployment with your Kubernetes tooling, or for example with:

```shell
kubectl wait --for=condition=available -n kubeflow deployment/model-registry-deployment --timeout=1m
kubectl logs -n kubeflow deployment/model-registry-deployment
```

Optionally, you can also manually forward the REST API container port of Model Registry and interact with the [REST API](reference/rest-api),
for example with:
```shell
kubectl port-forward svc/model-registry-service -n kubeflow 8081:8080
# in another terminal:
curl -X 'GET' \
  'http://localhost:8081/api/model_registry/v1alpha3/registered_models?pageSize=100&orderBy=ID&sortOrder=DESC' \
  -H 'accept: application/json' | jq
```

If you are not receiving a `2xx` response, it might be the case you are trying to consume a different version (`v1alphaX`) of the REST API than intended.

### Perform the check from within a Notebook

To check the connection to the Model Registry from a Notebook instead, start a Terminal from the Notebook environment, then you can dry-run the connection with the following command:

```
curl model-registry-service.kubeflow.svc.cluster.local:8080/api/model_registry/v1alpha3/registered_models
```

or, alternatively, with the following command:

```
wget -nv -O- model-registry-service.kubeflow.svc.cluster.local:8080/api/model_registry/v1alpha3/registered_models
```

If the command executes without any error, you will get a JSON response from Model Registry, indicating the connection and request was successful.

You can use the same commands in a Jupyter Notebook cell by prefixing the command with `!` (e.g.: `! curl ...`).

## Next steps

- Follow the [Getting Started](getting-started.md) guide to learn how to use Model Registry.
