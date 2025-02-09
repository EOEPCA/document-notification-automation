# Introduction

!!! warning
    Work in progress... :construction_worker:

The _Notification and Automation Building Block_ is designed to enable efficient communication and automated processes within EOEPCA+.

The Notification & Automation BB is designed to facilitate intra-Building-Block asynchronous communications. This means it allows different parts of the system to communicate with each other without needing to wait for responses, thereby improving efficiency and responsiveness.

It supports triggers that can initiate automated behaviour. These triggers can be based on external events (eg. events from object storage,  etc) or can be scheduled to occur at certain times.

It supports triggers that can initiate automated behaviour. These triggers can be based on external events (eg. events from object storage,  etc) or can be scheduled to occur at certain times.


## About the Notification and Automation Building Block
The Notification & Automation Building Block is composed out of two core components:

* **Notification Service:** This component is responsible for handling asynchronous event-based messaging within the system. It allows different Building Blocks to send and receive notifications or alerts about certain events, enabling them to react to changes or updates without direct interaction.  
* **Automation Service:** This component allows for the definition and execution of automated tasks based on specific triggers. These can be data-driven, where certain data conditions initiate an action, or scheduled, where actions are performed at predetermined times.

The Notification and Automation Building Block builds upon the KNative framework, primarly around the functionality offered by [KNative-Eventing](https://knative.dev/docs/eventing/).
KNative-Eventing manages events, allowing applications to interface with external event sources, register and subscribe to events, and filter them. It decouples event producers from consumers, providing a flexible event-driven architecture.