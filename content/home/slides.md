+++
weight = 1
+++

# Edge-Cloud continuum

## Opportunities and challenges

The growth of the **Internet of Things** (IoT) lead to the need of transferring, processing and storing an unpredictable amount of data

**Cloud computing** as an established technology for storing and processing data for a variety of application.
Has some limitation in _real-time_ and _low-latency_ applications.

**Fog computing** combines edge computing and cloud computing by utilizing _multiple layers_ of the computational infrastructure.

Exploiting the **edge-cloud continuum** enables many opportunities for the _IoT_ but also presents some open challenges.

---

# Frameworks and methodologies

Many proposal in the literature tries to handle the **edge-cloud continuum** problem.

{{% multicol %}}
{{% col %}}

#### Osmotic computing

{{< figure src="images/osmotic-architecture.png" width="75%" >}}

{{% /col %}}

{{% col %}}

#### DR-BIP & DReAM

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
- The thesis work is focused on "closing the gap" between _simulation_ and _real deployments_
- With the framework, the user can focus on the _business logic_ of the system demanding to the framework _platform-specific_ aspects
- The **pulverization** approach born in the _aggregate computing_ context, but the framework aims to be versatile enough to allow the pulverization also in non-aggregate systems

---

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

---

# Framework requirements

{{< multicol >}}
{{% col %}}
Main framework's features:

- **Simple & Clean API:** easy to use
- **Extensibility & customization:** custom user-defined components
- **Flexibility:** cope with different deployments strategies
- **Multi-platform:** multiple architectures

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
- **rabbitmq-platform:** communicator based on [RabbitMQ](https://www.rabbitmq.com/)

{{% /col %}}

{{< col >}}
{{< figure src="images/framework-architecture.svg" width="100%" >}}
{{< /col >}}

{{< /multicol >}}

_Modularity_ and _extensibility_ are the main features of the framework enabling an incremental use of it.

---

# Configuration DSL

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

To enable _intra-components communication_, the framework provides two concepts: **ComponentRef** and **Communicator**.

{{< figure src="images/componentref-communicator.svg" >}}

{{% multicol %}}
{{% col %}}

### ComponentRef

The **reference** to a component  _abstracting_ from its physical place.

Some _communication optimizations_ can be performed by the framework.

{{% /col %}}

{{% col %}}

### Communicator

Represents the _communication_ between components abstracting from the specific **protocol**.

In-Memory & [RabbitMQ](https://www.rabbitmq.com/) communicators

{{% /col %}}
{{% /multicol %}}

---

# Validation and Testing

**Unit** and **Integration** tests for:

- verifying **ComponentRef** and **Communicators** functional correctness
- verifying the configuration produced by _configuration DSL,_ intercepting invalid usage
- verifying the setup of **ComponentRef** and **Communicators** produced by _platform DSL_
- Test the frame as a whole by verifying the device's _cycle_.

Demos for **relevant scenarios** with physical devices:

- _Moisture soil:_ the "hello world" of the pulverization showing the device decomposition
- _hot-warm-cold_ game: main focus on device's communication and interaction

<!-- Lot of effort has been put in the **testing** and **validating** the framework.

An extensive test suite has been developed to ensure the functional correctness of the framework via _unit tests_ and _integration tests._
A special focus has been put on verifying invalid configurations and scenarios that should occur using the framework.

Validation has been performed using two **relevant scenarios**: moisture soil regulation and hot-warm-cold game.
With this two scenarios, the framework has been tested in a real environment.

{{% multicol %}}
{{% col class="text-center" %}}

{{< figure src="images/demo1-physical.svg" width="60%" >}}

_Moisture soil_ demo

{{% /col %}}

{{% col class="text-center" %}}

{{< figure src="images/demo2-physical.svg" width="80%" >}}

_Hot-warm-cold_ demo

{{% /col %}}
{{% /multicol %}} -->

---

# Conclusions

The framework try to tackle _edge-cloud continuum_ complexity in CPS by:

- providing _clean_ and _effective_ API with _multiplatform_ support
- promoting _reusability_ and _independency_ from underlying infrastructure
- allowing users to focus on _functional_ aspects
- leaving the management of _infrastructural_ aspects to the framework

**Testing** and **demos** has been used to validate the _functional_ operation and to prove its the _effectiveness_ in real-world scenarios.

## Future work

- **Multi-protocol:** best protocol based on device's capabilities and required QoS
- **Dynamism:** adapt the system opportunistically to changing requirements
- **Automatic deployment:** DevOps methodologies for automatic system deployment
- **Performance:** _latency_ and _throughput_ evaluation in different deployment strategy

---

# Demo

[Video in cui mostro demo crowd: da finire]

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
