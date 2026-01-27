# Quick Start

Quick start instructions - including installation, e.g. of a local instance.

!!! warning
    Work in progress... :construction_worker:

## Knative Operator Installation

EOEPCA+ Notification & Automation relies on Knative APIs and controllers. As a prerequisite, you must install the Knative Operator, which provides lifecycle management (install/upgrade/configure) for Knative components (e.g., Serving and Eventing) through Kubernetes Custom Resources (CRs).

### Prerequisites
 * A Kubernetes cluster running a Knative-supported Kubernetes version. 
 * _kubectl_ configured to access the target cluster with cluster-admin (or equivalent) permissions (CRDs + cluster-scoped RBAC will be installed).
 * Sufficient cluster resources (Knative guidance): single-node clusters typically need ≥ 6 vCPU / 6 GB RAM / 30 GB disk, multi-node clusters ≥ 2 vCPU / 4 GB RAM / 20 GB disk per node.  ￼

!!! info "Version pinning (recommended)."
    For reproducibility (and GitOps), pin the Operator version you install rather than using “latest” implicitly. The Knative install guide references the latest stable Operator release and the GitHub releases page.  ￼

### Install the Knative Operator

You can install the Operator using either Kubernetes manifests or Helm. Both are supported by Knative.

#### Option A: Install using Kubernetes manifests (recommended for GitOps)

Knative documentation shows the latest stable Operator manifest installation command as follows:


```bash
kubectl apply -f https://github.com/knative/operator/releases/download/knative-v1.20.1/operator.yaml
```

!!! note Latest Version
    `knative-v1.20.1` is the latest as of 27 January 2026. Should be replaced with the desired version. 

Recommended (parameterized) form:

```bash
export KNATIVE_OPERATOR_VERSION="knative-v1.20.1"
kubectl apply -f "https://github.com/knative/operator/releases/download/knative-v${KNATIVE_OPERATOR_VERSION}/operator.yaml"
```

!!! tip "Upgrading older Operator installs (CRD API migration)"
    Upgrading older Operator installs (CRD API migration). Knative notes that Operator 1.5 is the last version supporting CRDs with both v1alpha1 and v1beta1. If upgrading from older Operator installs, you may need to apply the post-install migration manifest before installing newer versions. 

#### Option B: Install using Helm

Knative also provides an official Helm chart repository for the Operator.

Add the Knative Operator Helm repository:
```bash
helm repo add knative-operator https://knative.github.io/operator
helm repo update
```

Install the Operator chart:
```bash 
helm install knative-operator \ 
--create-namespace \
--namespace knative-operator \
knative-operator/knative-operator
``` 

To inspect configurable values run:

```bash
helm show values -n knative-operator knative-operator/knative-operator
```

### Verify installation

To verify the Knative Operator installation, check that the `knative-operator` namespace is created and the Operator pod is running:

```bash
kubectl get pods -n knative-operator
```

!!! info "Additional installation information"
    For additional installation options and troubleshooting, see the [Knative Operator installation guide](https://knative.dev/docs/install/).


## Notification & Automation Installation



For installing using helm, first clone from notification-automation helm chart repository:
```bash
git pull git@github.com:mneagul/helm-charts-dev.git
```

Then navigate to the notification-automation chart directory and run the helm install/upgrade command:
```bash
cd helm-charts-dev/charts/notification-automation
helm upgrade --install na . \ 
--namespace na \
--create-namespace \ 
--set serving.install=true \ 
--set eventing.install=true
```

If running in a minikube can use:
```bash
helm upgrade --install na . -n na --create-namespace \
--set serving.install=true \
--set eventing.install=true \
--set webhookSource.enabled=true \
--set webhookSource.ingress.enabled=true \
--set webhookSource.ingress.className=nginx \
--set 'webhookSource.ingress.hosts[0].host=hooks.127.0.0.1.nip.io
```

In order to forward the ports for the webhook source, run the following command in a separate terminal:
```bash
kubectl port-forward --namespace na svc/webhook-source 8080:80
```

!!! tip "Known Issue"
    Due to timing issues in deployment, the Notification controller may fail to start properly on first install. If this happens, simply run the install command again.