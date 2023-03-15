+++
weight = 1
+++

# Overview

- **Edge-cloud continuum**: _stratification_ and _heterogeneity_ complicate the deployments
- **Pulverization** approach to tackle this complexity
- Fill the gap between _simulation_ and _physical_ deployments
- Design and implementation of a _framework_ to support the **pulverization** approach
- _Testing_ and _validation_ of the framework in real world scenarios
- _Improvements_ and _future directions_ of the framework

---

# Edge-Cloud continuum

- Edge-Cloud systems are _layered_ and _heterogeneous_
- They provide _opportunities_ but also _challenges_
- Several approaches have been proposed to manage this _complexity_

{{% multicol %}}
{{% col %}}

#### DR-BIP & DReAM

{{< figure src="images/motif-concept.png" caption="<b>Credits:</b> Rim El Ballouli et al. \"Four Exercises in Programming Dynamic Reconfigurable Systems: Methodology and Solution in DR-BIP\"" width="87%" >}}

{{% /col %}}

{{% col %}}

#### Osmotic Computing

{{< figure src="images/osmotic-architecture.png" caption="<b>Credits</b>: Massimo Villari et al. \"Osmosis: The Osmotic Computing Platform for Microelements in the Cloud, Edge, and Internet of Things\"" width="80%" >}}

{{% /col %}}
{{% /multicol %}}

---

# Pulverization approach

Pulverization _breaks down_ the system into _smaller computational units_ that are continuously executed across the available hosts.

{{% multicol %}}
{{% col %}}

{{< figure src="images/framework-components-interactions.svg" >}}

{{% /col %}}

{{% col %}}

{{< figure src="images/mape-cycle.svg" width="50%" >}}

{{% /col %}}
{{% /multicol %}}

This approach promotes the **deployment independence** of the system by separating _functional aspects_ from _deployment aspects_.

---

# Framework requirements

{{< multicol >}}
{{% col %}}
Main framework's features:

- **Simple & Clean API:** easy to use
- **Extensibility**: custom user-defined components
- **Flexibility**: cope with different deployments strategies
- **Multi-platform**: multiple architectures

{{% /col %}}

{{< col >}}
{{< figure src="images/kotlin-multiplatform.svg" >}}
{{< /col >}}

{{< /multicol >}}

The framework is written in [Kotlin Multiplatform](https://kotlinlang.org/docs/multiplatform.html) supporting:  
_JVM, Android, JS, iOS, Linux, macOS,_ and _Windows_.

---

# Framework architecture

{{< multicol >}}
{{% col %}}
The framework _modules:_

- **core**: the pulverization concepts
- **platform**: logic needed to implement a pulverized system
- **rabbitmq-platform**: communicator based on [RabbitMQ](https://www.rabbitmq.com/)

{{% /col %}}

{{< col >}}
{{< figure src="images/framework-architecture.svg" width="100%" >}}
{{< /col >}}

{{< /multicol >}}

_Modularity_ and _extensibility_ are the main features of the framework enabling an incremental use of it.

---

# Configuration DSL

The **system** can be defined via a dedicated _DSL_ to specify the structure of each _device_.  

```kotlin{|2-5|6-8|}
val configuration = pulverizationConfig {
    logicalDevice("device-1") {
        BehaviourComponent and CommunicationComponent deployableOn Cloud
        SensorsComponent and ActuatorsComponent deployableOn Device
    }
    logicalDevice("device-2") {
        CommunicationComponent and BehaviourComponent and ActuatorsComponent deployableOn Device
    }
}
```

Each **deployment unit** can be constructed in a _compositional_ and _declarative_ way.

```kotlin{|2-3|4-6|}
val platform = pulverizationPlatform(config.getDeviceConfiguration("device-1")!!) {
        behaviourLogic(SmartphoneBehaviour(), ::deviceBehaviourLogic)
        communicationLogic(SmartphoneCommunication(), ::deviceCommunicationLogic)
        withPlatform { RabbitmqCommunicator(hostname = "rabbitmq") }
        withRemotePlace { defaultRabbitMQRemotePlace() }
    }

platform.start().joinAll()
platform.stop()
```

---

# Components communication

{{< figure src="images/componentref-communicator.svg" >}}

{{% multicol %}}
{{% col %}}

<h3 class="text-center">ComponentRef</h3>

The **reference** to a component  _abstracting_ from its physical place.

Some _communication optimizations_ can be performed by the framework.

{{% /col %}}

{{% col %}}

<h3 class="text-center">Communicator</h3>

Represents the **communication** between components abstracting from the specific _protocol_.

In-Memory & [RabbitMQ](https://www.rabbitmq.com/) communicators

{{% /col %}}
{{% /multicol %}}

---

# Validation and Testing

_Unit_ and _integration_ tests for:

- Verifying functional correctness of **pulverization concepts** modelled in the framework
- Verifying the _DSL_ configuration, intercepting invalid usage
- Verifying the framework as a whole to be consistent with the **pulverization** formalization

Demos for **relevant scenarios** with physical devices:

- _Moisture soil_: the "hello world" of the pulverization showing the device decomposition
- _Hot-warm-cold_ game: main focus on device's communication and interaction

---

{{% section %}}

# Demo Architecture

Determine the _crowding_ of devices in a room using _Bluetooth LE_ and deploying the system through the **pulverization framework**.

{{% multicol %}}
{{% col %}}

<h4 class="text-center">Logical architecture</h3>

{{< figure src="images/demo-logical-devices.svg" >}}

{{% /col %}}

{{% col %}}

<h4 class="text-center">Physical architecture</h3>

{{< figure src="images/demo-physical-architecture.svg" >}}

{{% /col %}}
{{% /multicol %}}

<!-- ---

# Demo configuration

```kotlin
val config = pulverizationConfig {
    logicalDevice("estimator") {
        StateComponent and BehaviourComponent and
        ActuatorsComponent and CommunicationComponent deployableOn Device
    }
    logicalDevice("smartphone") {
        StateComponent and BehaviourComponent and
        SensorsComponent and CommunicationComponent deployableOn Device
    }
    logicalDevice("smartphone-offloaded") {
        StateComponent and BehaviourComponent deployableOn Cloud
        CommunicationComponent and SensorsComponent deployableOn Device
    }
}
```

---

# Demo platform

#### Estimator platform

```kotlin
val estimatorPlatform = pulverizationPlatform(config.getDeviceConfiguration("estimator")!!) {
    behaviourLogic(EstimatorBehaviour(), ::estimatorBehaviourLogic)
    stateLogic(StateComponent(), ::stateComponentLogic)
    communicationLogic(CommunicationComponent(), ::communicationComponentLogic)
    actuatorsLogic(EstimatorActuatorsContainer(), ::estimatorActuatorLogic)

    withPlatform { RabbitmqCommunicator("localhost") }
    withRemotePlace { defaultRabbitMQRemotePlace() }
}
```

#### Smartphone platform

```kotlin
val smartphonePlatform = pulverizationPlatform(config.getDeviceConfiguration("smartphone")!!) {
    // behaviourLogic(SmartphoneBehaviour(), ::smartphoneBehaviourLogic)
    // stateLogic(StateComponent(), ::stateComponentLogic)
    communicationLogic(CommunicationComponent(), ::communicationComponentLogic)
    actuatorsLogic(SmartphoneActuatorsContainer(), ::smartphoneActuatorLogic)

    withPlatform { RabbitmqCommunicator("localhost") }
    withRemotePlace { defaultRabbitMQRemotePlace() }
}
``` -->

{{% /section %}}

---

# Crowd Demo

<video controls loop width="85%" >
<source src="images/demo-pulverization-crowd.mp4" type="video/mp4" >
Your browser does not support the video tag.
</video>

---

# Conclusions

The framework try to tackle _edge-cloud continuum_ complexity in CPS by:

- Providing _clean_ and _effective_ API with _multiplatform_ support
- Promoting _reusability_ and _independency_ from underlying infrastructure
- Separating _functional_ aspects from _infrastructural_ ones

Testing and demos have been used to validate the _functional_ operation and to prove the _effectiveness_ with physical devices.

---

# Future work

- **Multi-protocol**: best protocol based on device's capabilities and required QoS
- **Dynamism**: adapt the system opportunistically to changing requirements
- **Automatic deployment**: DevOps methodologies for automatic system deployment
- **Performance**: _latency_ and _throughput_ evaluation in different deployment strategy

---

# References

{{% refentry fa-class="fa-brands fa-github" %}}

[`nicolasfara/pulverization-framework`](https://github.com/nicolasfara/pulverization-framework)

{{% /refentry %}}

{{% refentry fa-class="fa-brands fa-github" %}}

[`nicolasfara/pulverization-moisture-soil`](https://github.com/nicolasfara/pulverization-moisture-soil)

{{% /refentry %}}

{{% refentry fa-class="fa-brands fa-github" %}}

[`nicolasfara/pulverization-hot-warm-cold`](https://github.com/nicolasfara/pulverization-hot-warm-cold)

{{% /refentry %}}

{{% refentry fa-class="fa-brands fa-github" %}}

[`nicolasfara/pulverization-crowd`](https://github.com/nicolasfara/pulverization-crowd)

{{% /refentry %}}

{{% refentry fa-class="fa-solid fa-book" %}}

[`nicolasfarabegoli.it/pulverization-framework`](https://nicolasfarabegoli.it/pulverization-framework)

{{% /refentry %}}
