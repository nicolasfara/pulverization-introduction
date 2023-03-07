+++
weight = 1
+++

{{% section %}}

# Edge-Cloud continuum

## Opportunities and challenges

The growth of the **Internet of Things** (IoT) lead to the need of transferring, processing and storing an unpredictable amount of data.

**Cloud computing** as an established technology for storing and processing data for a variety of application.
Has some limitation in _real-time_ and _low-latency_ applications.

**Fog computing** combines edge computing and cloud computing by utilizing _multiple layers_ of the computational infrastructure.

Exploiting the **edge-cloud continuum** enables many opportunities for the _IoT_ but also presents some open challenges.

---

# Frameworks and methodologies

Many proposal in the literature tries to handle the **edge-cloud continuum** problem.

{{% multicol %}}
{{% col %}}

{{< figure src="images/osmotic-architecture.png" width="75%" >}}

{{% /col %}}

{{% col %}}

{{< figure src="images/motif-concept.png" width="80%" >}}

{{% /col %}}
{{% /multicol %}}

**Osmotic computing** main focus on distributed microservices architectures, while **DR-BIP** and **DReAM** focuses on the orchestration of distributed systems
by dynamically adapting to the changing requirements basing the system on the _motif_ concept.

---

# Pulverization approach

Modern **CPS** are increasingly large, heterogeneous and dynamics which makes challenging engineering systems that can exploit _opportunistically_ the available resources.

**Pulverization** address this problem by breaking down the system into _smaller computational pieces_ that are continuously executed across the available hosts.

These **sub-components** can be deployed and wired separately, allowing for a _separation of concerns_ between the business logic and the deployment aspects.

---

# Motivation

- Actually the **pulverization** approach is tested only in simulation
- The thesis work is focused on "close the gap" between _simulation_ and _real deployments_
- With the framework, the user can focus on the _business logic_ of the system demanding to the framework _platform-specific_ aspects
- The **pulverization** approach born in the _aggregate computing_ context, but the framework aims to be versatile enough to allow the pulverization also in non-aggregate systems

{{% /section %}}

---

{{% section %}}

# Pulverization domain model

A **logical device** represents a device in the system where its components can be deployed _independently_ on the available infrastructure.  

{{% multicol %}}
{{% col %}}

A logical device is decomposed into:

- **Behaviour**: the logic of the device
- **Communication**: the way the device communicates with other devices
- **State**: the state of the device
- **Sensors**: the sensors of the device
- **Actuators**: the actuators of the device

{{% /col %}}

{{% col %}}

{{< figure src="images/framework-components-interactions.svg" >}}

{{% /col %}}
{{% /multicol %}}

A _host_ represent the physical device where the components are deployed.  
We refer with **thin host** to a host with limited resources (e.g. embedded devices).  
A **thick host** represents a host with an high computational power (e.g. cloud).

---

# Device's components interaction

Each device performs a _MAPE-like_ cycle to achieve the device's behaviour:

{{% multicol %}}
{{% col %}}

1. _Context acquisition:_ get **sensors** and **communication** data storing them in the **state**
2. _Computation:_ compute the **behaviour** function
3. _Coordination data propagation:_ coordination data is sento to all the neighbours
4. _Actuation:_ the **actuators** are activated according to the prescriptive actions
{{% /col %}}

{{% col %}}

{{< figure src="images/mape-cycle.svg" width="58%" >}}

{{% /col %}}
{{% /multicol %}}

{{% /section %}}

---

{{% section %}}

# Framework requirements

{{< multicol >}}
{{% col %}}
Main framework's features:

- **Simple & Clean API:** simplifying the development of the system
- **Extensibility & customization:** the user can customize the framework to fit its needs
- **Flexibility:** supports different deployment strategies and scenarios
- **Multiplatform:** support a wide range of platform and architectures enabling a wide adoption of the framework

{{% /col %}}

{{< col >}}
{{< figure src="images/kotlin-multiplatform.svg" >}}
{{< /col >}}

{{< /multicol >}}

The framework is entirely written in **Kotlin** and it is based on the [Kotlin Multiplatform](https://kotlinlang.org/docs/multiplatform.html)
technology supporting _JVM, Android, JS, iOS, Linux, macOS,_ and _Windows_.

<!--
# Demo 1

## Single device multiple components

The first demo shows how to "pulverize" a simple device for regulating the moisture of a terrain.
{{< multicol >}}
{{% col %}}

1. The _sensor_ component collect the moisture level of the terrain
2. The _behaviour_ component decides if the _actuator_ component should be activated or not based on the threshold defined in the _state_
3. The _actuator_ component open or close the valve based on the information given by the _behaviour_ component

{{% /col %}}

{{< col >}}
{{< figure src="images/demo-1.svg" width="80%" >}}
{{< /col >}}

{{< /multicol >}}

---

# Demo 2

## Multiple device multiple components

This demo tries to emulate the _hot-warm-cold_ game using two smartphone and one antenna.

Smartphones search for the antenna by measuring its **distance** (via Bluetooth RSSI) and sharing that value with other smartphones.
The raspberry receives information about the distances of smartphones and emits light that is as bright the closer the devices are and less bright the
farther away they are. In this case, the actuator for the raspberry is an LED.

{{< multicol >}}
{{< col >}}
{{< figure src="images/demo-2-logical-comm.svg" width="55%" >}}
{{< /col >}}

{{< col >}}
{{< figure src="images/demo-2-physical.svg" width="60%" >}}
{{< /col >}}
{{< /multicol >}}

-->

---

# Framework architecture

{{< multicol >}}
{{% col %}}
The framework _modules:_

- **core**: defines the pulverization concepts and interfaces needed to implement a pulverized system
- **platform**: manages all the logic needed to implement a pulverized system
- **rabbitmq-platform:** implementation of a communicator based on [RabbitMQ](https://www.rabbitmq.com/)

{{% /col %}}

{{< col >}}
{{< figure src="images/framework-architecture.svg" width="100%" >}}
{{< /col >}}

{{< /multicol >}}

_Modularity_ and _extensibility_ are the main features of the framework enabling an incremental use of it.

---

# Configuration DSL

<!-- The configuration in the framework defines how many logical devices are present in the system and defines how a logical device is composed. -->

The **devices** configuration can be achieved using the following _DSL_:

```kotlin{|2-5|7-9|}
val configuration = pulverizationConfig {
    logicalDevice("device-1") {
        BehaviourComponent and CommunicationComponent deployableOn Cloud
        SensorsComponent and ActuatorsComponent deployableOn Device
    }

    logicalDevice("device-2") {
        CommunicationComponent and BehaviourComponent and ActuatorsComponent deployableOn Cloud
    }
}
```

---

# Platform DSL

The _DSL_ enables the user to configure the actual platform in a pure **declarative** fashion.

```kotlin{|3-5|7-8|}
suspend fun main() = coroutineScope {
    val platform = pulverizationPlatform(config.getDeviceConfiguration("smartphone")!!) {
            behaviourLogic(SmartphoneBehaviour(), ::smartphoneBehaviourLogic)
            stateLogic(SmartphoneState(), ::smartphoneStateLogic)
            communicationLogic(SmartphoneCommunication(), ::smartphoneCommunicationLogic)

            withPlatform { RabbitmqCommunicator(hostname = "rabbitmq") }
            withRemotePlace { defaultRabbitMQRemotePlace() }
        }

    platform.start().joinAll()
    platform.stop()
}
```

---

# Components communication

To enable a seamless _intra-components communication_, the framework provides two concepts: **ComponentRef** and **Communicator**.

{{% multicol %}}
{{% col %}}

### ComponentRef

Models the **reference** to a component in the system _abstracting_ from the physical place where the component is deployed.

With this abstraction _communication optimizations_ can be performed by the framework.

{{% /col %}}

{{% col %}}

### Communicator

Represents the way _how components communicate_ with each other abstracting from the specific **protocol**.

The framework provides a default implementation based on [RabbitMQ](https://www.rabbitmq.com/) but other implementations can be provided.

{{% /col %}}
{{% /multicol %}}

---

# Validation and Testing

Lot of effort has been put in the **testing** and **validating** the framework.

An extensive test suite has been developed to ensure the functional correctness of the framework via _unit tests_ and _integration tests._
A special focus has been put on verifying invalid configurations and scenarios that should occur using the framework.

Validation has been performed using two **relevant scenarios**: moisture soil regulation and hot-warm-cold game.
With this two scenarios, the framework has been tested in a real environment.

{{% multicol %}}
{{% col %}}

{{< figure src="images/demo1-physical.svg" width="60%" >}}

{{% /col %}}

{{% col %}}

{{< figure src="images/demo2-physical.svg" width="80%" >}}

{{% /col %}}
{{% /multicol %}}

---

# Demo

{{% /section %}}

---

# Future work

---

# References

- [Pulverization framework] [https://github.com/nicolasfara/pulverization-framework](https://github.com/nicolasfara/pulverization-framework)
- [Demo 1] [https://github.com/nicolasfara/pulverization-moisture-soil](https://github.com/nicolasfara/pulverization-moisture-soil)
- [Demo 2] [https://github.com/nicolasfara/pulverization-hot-warm-cold](https://github.com/nicolasfara/pulverization-hot-warm-cold)
