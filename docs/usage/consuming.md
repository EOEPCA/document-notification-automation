# Consuming Events

In EOEPCA+ notification events consumers can be any URL addressable `Sink`, both internal and external.

Consumers can register to events using two approaches:

- Creating a trigger
- Subscribing to a channel

Registering to events using a trigger can be achieved by using the cli:

```bash
kn trigger create on-resource-added --broker primary \
    --filter type=org.eoepca.resource-registration.resource.added \
    -s http://my-service.default.svc.cluster.local
```

The equivalent registration using a kubernetes manifest can be achived with:

```yaml
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  labels:
    eventing.knative.dev/broker: primary
  name: on-resource-added
  namespace: default
spec:
  broker: primary
  filter:
    attributes:
      type: org.eoepca.resource-registration.resource.added
  subscriber:
    uri: http://my-service.default.svc.cluster.local
```

After the registration the HTTP endpoint will receive CloudEvents using `HTTP POST`, with attributes encoded as `HTTP` headers and the event as the POST body.