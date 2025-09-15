Installation of the N&A BB involves installing the required backend software and the default N&A Configuration.

## Requirements

### Supported KNative versions


| KNative Release      | Date       | EOL.            | Min K8s Version | Notes |
|---------| ---------- |------------|-----------------| ----- |
| 1.19    | 2025-07-22 | 2026-01-28 | 1.32            |       |
| 1.18    | 2025-04-22 | 2025-10-28 | 1.31            |       |

## Installation

Both helm chart and manuall installation require some of the prerequisetes to be satisfied:
- A recent K8S version based on the requirements matrix
- KNative Eventing and, optionally but highly recommanded, KNative serving installed and configured.

### KNative-Operator

The primary components required is the `knative-operator`. It can be installed manually:

```bash
helm repo add knative-operator https://knative.github.io/operator
helm install knative-operator --create-namespace --namespace knative-operator knative-operator/knative-operator
```

In a few seconds after installing you should be able to see the required KNative-Operator components running in the `knative-operator` namespace:

```
kubectl get pods -n knative-operator
NAME                                READY   STATUS    RESTARTS   AGE
knative-operator-5c567b8ccf-h2wmg   1/1     Running   0          64s
operator-webhook-5846477f49-p6tj2   1/1     Running   0          64s
```

### Helm Chart based Installation

Assuming that the Knative Operator is deployed the N&A can be deployed as follows:

```
helm upgrade --install na notification-automation \
    --namespace na \
    --create-namespace \
    --set knativeEventing.install=true \
    --set authenticationOidc=true
```

- `knativeEventing.install=true`: installs the KNativeEventing in the `knative-eventing` namespace using the default EOEPCA configuration
- `authenticationOidc=true`: enabled internal OIDC authentication



### Manual Installation

#### KNative-Eventing

KNative-Eventing can be installed using the default `in-memory` backend by 
creating an install of the `KnativeEventing` CRD:

```yaml
apiVersion: operator.knative.dev/v1beta1
kind: KnativeEventing
metadata:
  name: knative-eventing # Mandatory
  namespace: knative-eventing # Mandatory
spec:
  config:
    config-features:
      authentication-oidc: enabled
      eventtype-auto-create: enabled
    default-ch-webhook:
        default-ch-config: |
            clusterDefault:
                apiVersion: messaging.knative.dev/v1
                kind: InMemoryChannel
                spec:
                    delivery:
                        backoffDelay: PT0.5S
                        backoffPolicy: exponential
                        retry: 5
  version: 1.18.1
```

This approach install KNative Eventing with the default `in-memory` backend.

## Optional Components

The KNative deployment can be suplemented by additonal, optional components like Camel-K. This components have the merit of adding more integration functionality.

### Camel-K


```sh
helm repo add camel-k https://apache.github.io/camel-k/charts/
helm install camel-k camel-k/camel-k -n camel-k
```