# Installation and Configuration

The Helm chart for **Notification & Automation** is designed to simplify the deployment of Knative components required for event-driven architectures. This document provides detailed instructions on installing and configuring the chart.
The chart can be found in:
!!! info "Notification & Automation Helm Chart Repository"
    https://github.com/mneagul/helm-charts-dev/tree/na-helm-refactored/charts/notification-automation


Helm chart that bootstraps **Knative Eventing** and/or **Knative Serving** using the **Knative Operator**, and (optionally) creates a **default Knative Broker** in the Helm release namespace.

This chart is intentionally small: it mainly creates namespaces, Knative Operator custom resources (`KnativeEventing`, `KnativeServing`), and a `Broker`. Additionally it can deploy helper workloads:
    - `webhookSource`: a webhook receiver that can be **SinkBound** to a Broker (or other sink)
    - `cloudEventsPlayer`: a UI/service that can subscribe to a Broker via a Knative Trigger

---

## What this chart installs

Depending on values you set, the chart can install:

- A `Namespace` for Knative Eventing (`templates/knative-eventing-namespace.yaml`)
- A `KnativeEventing` custom resource (`templates/knative-eventing.yaml`)
- A `Namespace` for Knative Serving (`templates/knative-serving-namespace.yaml`)
- A `KnativeServing` custom resource (`templates/knative-serving.yaml`)
- A Knative `Broker` (default broker) in the **release namespace** (`templates/default-broker.yaml`)

It also includes an optional pre-flight check for Knative Operator CRDs (`templates/knative-crd-check.yaml`).

### Optional workloads

- Webhook Source Deployment/Service/Ingress, plus optional ConfigMap/Secret/SinkBinding (`templates/webhook-source*.yaml`)
- CloudEvents Player Deployment/Service/Ingress, plus optional Trigger (`templates/cloudevents-player*.yaml`)

---

## Prerequisites

- Kubernetes **>= 1.31.0** (see `Chart.yaml`)
- Helm 3.x
- If you enable CRD enforcement (`crdChecks.enforce=true`), Helm must be able to reach the cluster because the chart uses Helm’s `lookup` function.

### Knative Operator

This chart declares a dependency on:

- `knative-operator` chart from `https://knative.github.io/operator` at version `v1.19.2` (see `requirements.yaml` / `requirements.lock`).

The dependency is used to install the **Knative Operator**, which then reconciles the `KnativeEventing` and `KnativeServing` resources created by this chart.

## Install / Upgrade

From the repository root:

```bash
helm dependency update charts/notification-automation
```

### Install with defaults

By default, both Eventing and Serving are disabled (`eventing.install=false`, `serving.install=false`).

```bash
helm upgrade --install notification-automation charts/notification-automation \
  -n <your-namespace> --create-namespace
```

!!! note "Broker creation"
    By default a preconfigured broker is deployed and a simple topology is generated for it.

### Install Knative Eventing

```bash
helm upgrade --install notification-automation charts/notification-automation \
  -n <your-namespace> --create-namespace \
  --set eventing.install=true
```

### Install Knative Eventing and create a default Broker

A default Broker created when **Eventing is installed** and broker creation is enabled.

```bash
helm upgrade --install notification-automation charts/notification-automation \
  -n <your-namespace> --create-namespace \
  --set eventing.install=true \
  --set eventing.defaultBroker.create=true
```
!!! info "Broker namespace"
    The Broker is created in the **Helm release namespace** (`.Release.Namespace`), not in `eventing.namespace`.

### Install Knative Serving

```bash
helm upgrade --install notification-automation charts/notification-automation \
  -n <your-namespace> --create-namespace \
  --set serving.install=true
```

### Install both Eventing and Serving

```bash
helm upgrade --install notification-automation charts/notification-automation \
  -n <your-namespace> --create-namespace \
  --set eventing.install=true \
  --set serving.install=true
```

## Configuration

All configuration is done via `values.yaml`.

### CRD checks (optional enforcement)

File: `templates/knative-crd-check.yaml`

If you set:

```yaml
crdChecks:
  enforce: true
```

Then, when you enable Eventing and/or Serving, Helm will verify the corresponding Knative Operator CRD exists:

- `knativeeventings.operator.knative.dev`
- `knativeservings.operator.knative.dev`

If the CRD is missing, the release fails early with a clear error.

!!! notes "Notes"
    - Keep this `false` when running `helm template` without a live cluster.
    - If Knative Operator is not installed yet, the CRD check will fail. Ensure the Operator is installed before enabling this check.

### Knative Eventing

Enable Eventing:

```yaml
eventing:
  install: true
```

The chart creates:

- Namespace `eventing.namespace` (default: `knative-eventing`)
- `KnativeEventing` resource named `knative-eventing` in that namespace

Key behaviors from `templates/knative-eventing.yaml`:

- Feature flags:
    - `authentication-oidc` is `enabled` when `eventing.authenticationOidc=true`, otherwise `disabled`
    - `eventtype-auto-create` is always `enabled`
- Default channel webhook config is set so that `InMemoryChannel` is the cluster default with delivery retry/backoff:
    - `backoffDelay: PT0.5S`
        - `backoffPolicy: exponential`
          - `retry: 5`
- Operator target version is taken from `eventing.version` (default: `"1.18.1"`)

### Default Broker

When both are true:

```yaml
eventing:
  install: true
  defaultBroker:
    create: true
```

The chart will create:

- `Broker` named `eventing.defaultBroker.name` (default: `default`)
- In the **release namespace** (`.Release.Namespace`)

### Knative Serving

Enable Serving:

```yaml
serving:
  install: true
```

The chart creates:

- Namespace `serving.namespace` (default: `knative-serving`)
- `KnativeServing` resource named `knative-serving` in that namespace

Key behaviors from `templates/knative-serving.yaml`:

- Optional domain configuration:
    - If `serving.domain` exists and is non-empty, it sets:

        ```yaml
        spec:
          config:
            domain:
              <serving.domain>: ""
        ```

- Networking:
    - `ingress-class: kourier.ingress.networking.knative.dev`
      - Kourier is enabled via:

        ```yaml
        spec:
          ingress:
            kourier:
              enabled: true
        ```

- Operator target version is taken from `serving.version` (default: `"1.18.0"`)

---

## Webhook Source (optional)

Templates:
- `templates/webhook-source.yaml`
- `templates/webhook-source-ingress.yaml`

Enable:

```yaml
webhookSource:
  enabled: true
```

This renders a Deployment (and optionally Service/Ingress) for a webhook receiver image.

### Environment variables

The Deployment sets:

- `PORT` (from `webhookSource.containerPort`)
- `OIDC_TOKEN_PATH` (from `webhookSource.oidcTokenPath`)
- `GITHUB_WEBHOOK_SECRET` and `GITLAB_WEBHOOK_SECRET` (from an optional Secret)
- `PROJECTS_CONFIG` (from an optional ConfigMap)
- `webhookSource.extraEnv` (verbatim)

### ServiceAccount

- Created when `webhookSource.serviceAccount.create=true`
- Name is `webhookSource.serviceAccount.name` if set, otherwise defaults to `webhookSource.name`

### Service

Created when `webhookSource.service.create=true`.

### Ingress

Created when all of these are true:

- `webhookSource.enabled=true`
- `webhookSource.service.create=true`
- `webhookSource.ingress.enabled=true`

Ingress supports:

- `webhookSource.ingress.className`
- `webhookSource.ingress.annotations`
- `webhookSource.ingress.tls`
- `webhookSource.ingress.hosts[].host` + `hosts[].paths[]`

### ConfigMap (optional)

If `webhookSource.configMap.create=true`, creates a ConfigMap named `webhookSource.configMap.name` with one key (`webhookSource.configMap.key`) containing `webhookSource.configMap.projectsJson`.

### Secret (optional)

If `webhookSource.secrets.create=true`, creates a Secret named `webhookSource.secrets.name` with keys:

- `webhookSource.secrets.githubKey` containing `webhookSource.secrets.githubSecret`
- `webhookSource.secrets.gitlabKey` containing `webhookSource.secrets.gitlabSecret`

### SinkBinding (optional)

A `SinkBinding` is created when either:

- `webhookSource.sinkBinding.create=true`, **or**
- Eventing + default broker are enabled (`eventing.install=true` and `eventing.defaultBroker.create=true`)

If you enable it explicitly (`webhookSource.sinkBinding.create=true`), you can configure the sink as either:

- a reference (`webhookSource.sinkBinding.sink.ref`) **and/or**
- a URI (`webhookSource.sinkBinding.sink.uri`)

If you do **not** enable it explicitly, but you *do* enable the default broker, the chart will automatically bind the Deployment to:

- `Broker` named `eventing.defaultBroker.name`

---

## CloudEvents Player (optional)

Templates:
- `templates/cloudevents-player.yaml`
- `templates/cloudevents-player-ingress.yaml`

Enable:

```yaml
cloudEventsPlayer:
  enabled: true
```

This renders a Deployment (and optionally Service/Ingress) for a CloudEvents Player UI.

### Trigger subscription (optional)

If `cloudEventsPlayer.subscribeBroker.enabled=true`, the chart may create a Knative Trigger that routes events from a Broker to the CloudEvents Player Service.

Broker resolution rules (`templates/cloudevents-player.yaml`):

- If `cloudEventsPlayer.subscribeBroker.broker.name` is set, it uses that.
- Otherwise, if Eventing + default broker are enabled, it uses `eventing.defaultBroker.name`.
- If it can’t determine a broker name, it won’t create a Trigger.

### Ingress

Created when all of these are true:

- `cloudEventsPlayer.enabled=true`
- `cloudEventsPlayer.service.create=true`
- `cloudEventsPlayer.ingress.enabled=true`

---


## Values reference (key settings)

### Knative

| Key | Type | Default | Description |
|---|---:|---|---|
| `crdChecks.enforce` | bool | `false` | Fail early if Knative Operator CRDs are missing (uses `lookup`) |
| `eventing.install` | bool | `false` | Install Knative Eventing |
| `eventing.version` | string | `"1.18.1"` | Knative Eventing version passed to operator |
| `eventing.namespace` | string | `knative-eventing` | Namespace to install `KnativeEventing` into |
| `eventing.authenticationOidc` | bool | `true` | Toggle `authentication-oidc` feature flag |
| `eventing.defaultBroker.create` | bool | `true` | Create a Broker in the release namespace (only if Eventing install is enabled) |
| `eventing.defaultBroker.name` | string | `default` | Broker name |
| `serving.install` | bool | `false` | Install Knative Serving |
| `serving.version` | string | `"1.18.0"` | Knative Serving version passed to operator |
| `serving.namespace` | string | `knative-serving` | Namespace to install `KnativeServing` into |
| `serving.domain` | string | `functions.eoepca.org` | Optional domain mapping entry (only if non-empty) |

### Webhook Source

| Key | Type | Default | Description |
|---|---:|---|---|
| `webhookSource.enabled` | bool | `false` | Deploy the webhook receiver workload |
| `webhookSource.name` | string | `webhook-source` | Base name for K8s resources |
| `webhookSource.image.repository` | string | `ghcr.io/eoepca/na-webhook-source` | Image repository (required if enabled) |
| `webhookSource.image.tag` | string | `sha-895c85f` | Image tag |
| `webhookSource.service.create` | bool | `true` | Create Service |
| `webhookSource.ingress.enabled` | bool | `false` | Create Ingress (requires Service) |
| `webhookSource.configMap.create` | bool | `false` | Create ConfigMap for projects JSON |
| `webhookSource.secrets.create` | bool | `false` | Create Secret containing webhook shared secrets |
| `webhookSource.sinkBinding.create` | bool | `false` | Create SinkBinding to attach sink env to the Deployment |

### CloudEvents Player

| Key | Type | Default | Description |
|---|---:|---|---|
| `cloudEventsPlayer.enabled` | bool | `false` | Deploy cloudevents-player |
| `cloudEventsPlayer.name` | string | `cloudevents-player` | Base name for K8s resources |
| `cloudEventsPlayer.image.repository` | string | `quay.io/ruben/cloudevents-player` | Image repository (required if enabled) |
| `cloudEventsPlayer.image.tag` | string | `v1.3` | Image tag |
| `cloudEventsPlayer.service.create` | bool | `true` | Create Service |
| `cloudEventsPlayer.ingress.enabled` | bool | `false` | Create Ingress (requires Service) |
| `cloudEventsPlayer.subscribeBroker.enabled` | bool | `true` | Create Trigger subscribing to a Broker (if broker name can be determined) |
| `cloudEventsPlayer.subscribeBroker.broker.name` | string | `""` | Broker name override |

!!! info "Notes/gotchas"
    - The default Broker is created in the **namespace where you installed the Helm release**, not in `knative-eventing`.
    - If you run `helm template` with `crdChecks.enforce=true`, rendering can fail because `lookup` can’t query CRDs without cluster access.
    - This chart does not deploy any application workload itself; it’s focused on bringing up Knative building blocks.
    - OIDC authentication for Knative Eventing is disabled by default.
