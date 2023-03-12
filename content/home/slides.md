+++
weight = 1
+++

# Overview

- **Edge-cloud continuum** in CPS: _stratification_ and _heterogeneity_
- **Pulverization** approach to tackle this complexity
- Fill the gap between _simulation_ and _real world_
- Design and implementation of a **framework** to support the **pulverization** approach
- _Testing_ and _validation_ of the framework in real world scenarios
- _Improvements_ and _future directions_ of the framework

---

# Edge-Cloud continuum

- Edge-Cloud systems are _layered_ and _heterogeneous_
- They provide _opportunities_ but also _challenges_
- Several approaches have been proposed to manage this **complexity**

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

This approach promotes the _deployment independence_ of the system by separating **functional aspects** from **deployment aspects**.

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

<video controls loop width="85%" >
<source src="images/demo-pulverization-crowd.mp4" type="video/mp4" >
Your browser does not support the video tag.
</video>

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
