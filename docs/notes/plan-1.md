# Notification & Automation Status Update and Plan


## Summary
Currently the ongoing work was split in the following:

1. Catch-up with the latest changes in KNative
1. Authentication / Authorization Work
1. Support other BBs in producing Events
1. Deployment update

This work is required for supporting the E2E scenarios. To this purpose we supported the Resource Discovery Building Block in adding the required functionality. Particuarly this resulted in the addition of a notification mechanism added to the `pgstac` backed and a PoC which constituted the base of the change.

Previously we supported the Processing BB in adding notification support to Zoo which is already integrated but will probably need to be updated with standardised events based in changes in the PubSub spec.

Also we supported the resource health BB in generating events and adding support for Authorization/Authentication (as they run potentially untrusted code). The authorization/authentication part is depended on more recent K8S versions.


## Event Producers

### eoAPI
Ar previously mentioned the required functionality was added to `pgstac` [^1] and a working PoC was developed [^2].

Required work on this side is the extension of the eoAPI Helm Chart with a new deployment listening to the `pgstac` notifications and dispatch to the configured `broker`. In the following days we will decide if NA BB adapts the PoC for this or it will be handled by Development Seed.

Pushing this was a priority as it makes any E2E functionality/demonstration real.

### Zoo

Gerald contributed initial functionality for this and it is already available in the code base.

## Resource Health

We had lots of discussions with the Resource Health people regarding this. At this point the event generation functionality should be close its final stages but will be dependent on KNative 1.17/1.18 with OIDC support.

## Resource Registration

Integration with this component is not yet started. We are investigating ways to proceed and personally I hope to address this during the BiDS code sprint with Angelos.

## Event consumers

### Documentation updates
As mentioned during the status update meeting we are working on the update of the documentation. This includes both usage instructions and deployment instructions + scripts.

### Serverless functionality

This part was affected by some changes in the SDK provided by the KNative Community.
At this point we consider the we can provided examples for deploying serverless functionality at least for Python and Go. A more streamlined experience is available using KNative Serving 1.18.

For now we aim to provide, for the first official release, KNative Serving examples based on recent BuildPacks and a dedicated EOEPCA repository.

It is worth mentioning that in 1.17 Knative added functionality for long running serverless functions (backed by JobSink's).

## UI

KNative does not provide any official UI for managing functions. To this end we are in the process of recruiting a frontend developer for creating a MVP and, eventually, a full blown interface.


## End to End Functionality





[^1] https://github.com/stac-utils/pgstac/pull/383
[^2] https://github.com/EOEPCA/resource-discovery/issues/167




