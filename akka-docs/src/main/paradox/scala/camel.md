# Camel

@@@ warning

Akka Camel is deprecated in favour of [Alpakka](https://github.com/akka/alpakka) , the Akka Streams based collection of integrations to various endpoints (including Camel).

@@@

## Introduction

The akka-camel module allows Untyped Actors to receive
and send messages over a great variety of protocols and APIs.
In addition to the native Scala and Java actor API, actors can now exchange messages with other systems over large number
of protocols and APIs such as HTTP, SOAP, TCP, FTP, SMTP or JMS, to mention a
few. At the moment, approximately 80 protocols and APIs are supported.

### Apache Camel

The akka-camel module is based on [Apache Camel](http://camel.apache.org/), a powerful and light-weight
integration framework for the JVM. For an introduction to Apache Camel you may
want to read this [Apache Camel article](http://architects.dzone.com/articles/apache-camel-integration). Camel comes with a
large number of [components](http://camel.apache.org/components.html) that provide bindings to different protocols and
APIs. The [camel-extra](http://code.google.com/p/camel-extra/) project provides further components.

### Consumer

Usage of Camel's integration components in Akka is essentially a
one-liner. Here's an example.

@@snip [Introduction.scala]($code$/scala/docs/camel/Introduction.scala) { #Consumer-mina }

The above example exposes an actor over a TCP endpoint via Apache
Camel's [Mina component](http://camel.apache.org/mina2.html). The actor implements the endpointUri method to define
an endpoint from which it can receive messages. After starting the actor, TCP
clients can immediately send messages to and receive responses from that
actor. If the message exchange should go over HTTP (via Camel's Jetty
component, only the actor's endpointUri method must be changed.

@@snip [Introduction.scala]($code$/scala/docs/camel/Introduction.scala) { #Consumer }

### Producer

Actors can also trigger message exchanges with external systems i.e. produce to
Camel endpoints.

@@snip [Introduction.scala]($code$/scala/docs/camel/Introduction.scala) { #imports #Producer }

In the above example, any message sent to this actor will be sent to
the JMS queue `orders`. Producer actors may choose from the same set of Camel
components as Consumer actors do.

### CamelMessage

The number of Camel components is constantly increasing. The akka-camel module
can support these in a plug-and-play manner. Just add them to your application's
classpath, define a component-specific endpoint URI and use it to exchange
messages over the component-specific protocols or APIs. This is possible because
Camel components bind protocol-specific message formats to a Camel-specific
[normalized message format](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/Message.java). The normalized message format hides
protocol-specific details from Akka and makes it therefore very easy to support
a large number of protocols through a uniform Camel component interface. The
akka-camel module further converts mutable Camel messages into immutable
representations which are used by Consumer and Producer actors for pattern
matching, transformation, serialization or storage. In the above example of the Orders Producer,
the XML message is put in the body of a newly created Camel Message with an empty set of headers.
You can also create a CamelMessage yourself with the appropriate body and headers as you see fit.

### CamelExtension

The akka-camel module is implemented as an Akka Extension, the `CamelExtension` object.
Extensions will only be loaded once per `ActorSystem`, which will be managed by Akka.
The `CamelExtension` object provides access to the @extref[Camel](github:akka-camel/src/main/scala/akka/camel/Camel.scala) trait.
The @extref[Camel](github:akka-camel/src/main/scala/akka/camel/Camel.scala) trait in turn provides access to two important Apache Camel objects, the [CamelContext](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/CamelContext.java) and the `ProducerTemplate`.
Below you can see how you can get access to these Apache Camel objects.

@@snip [Introduction.scala]($code$/scala/docs/camel/Introduction.scala) { #CamelExtension }

One `CamelExtension` is only loaded once for every one `ActorSystem`, which makes it safe to call the `CamelExtension` at any point in your code to get to the
Apache Camel objects associated with it. There is one [CamelContext](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/CamelContext.java) and one `ProducerTemplate` for every one `ActorSystem` that uses a `CamelExtension`.
By Default, a new [CamelContext](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/CamelContext.java) is created when the `CamelExtension` starts. If you want to inject your own context instead,
you can extend the @extref[ContextProvider](github:akka-camel/src/main/scala/akka/camel/ContextProvider.scala) trait and add the FQCN of your implementation in the config, as the value of the "akka.camel.context-provider".
This interface define a single method `getContext` used to load the [CamelContext](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/CamelContext.java).

Below an example on how to add the ActiveMQ component to the [CamelContext](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/CamelContext.java), which is required when you would like to use the ActiveMQ component.

@@snip [Introduction.scala]($code$/scala/docs/camel/Introduction.scala) { #CamelExtensionAddComponent }

The [CamelContext](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/CamelContext.java) joins the lifecycle of the `ActorSystem` and `CamelExtension` it is associated with; the [CamelContext](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/CamelContext.java) is started when
the `CamelExtension` is created, and it is shut down when the associated `ActorSystem` is shut down. The same is true for the `ProducerTemplate`.

The `CamelExtension` is used by both *Producer* and *Consumer* actors to interact with Apache Camel internally.
You can access the `CamelExtension` inside a *Producer* or a *Consumer* using the `camel` definition, or get straight at the *CamelContext* using the `camelContext` definition.
Actors are created and started asynchronously. When a *Consumer* actor is created, the *Consumer* is published at its Camel endpoint (more precisely, the route is added to the [CamelContext](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/CamelContext.java) from the [Endpoint](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/Endpoint.java) to the actor).
When a *Producer* actor is created, a [SendProcessor](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/processor/SendProcessor.java) and [Endpoint](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/Endpoint.java) are created so that the Producer can send messages to it.
Publication is done asynchronously; setting up an endpoint may still be in progress after you have
requested the actor to be created. Some Camel components can take a while to startup, and in some cases you might want to know when the endpoints are activated and ready to be used.
The @extref[Camel](github:akka-camel/src/main/scala/akka/camel/Camel.scala) trait allows you to find out when the endpoint is activated or deactivated.

@@snip [Introduction.scala]($code$/scala/docs/camel/Introduction.scala) { #CamelActivation }

The above code shows that you can get a `Future` to the activation of the route from the endpoint to the actor, or you can wait in a blocking fashion on the activation of the route.
An `ActivationTimeoutException` is thrown if the endpoint could not be activated within the specified timeout. Deactivation works in a similar fashion:

@@snip [Introduction.scala]($code$/scala/docs/camel/Introduction.scala) { #CamelDeactivation }

Deactivation of a Consumer or a Producer actor happens when the actor is terminated. For a Consumer, the route to the actor is stopped. For a Producer, the [SendProcessor](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/processor/SendProcessor.java) is stopped.
A `DeActivationTimeoutException` is thrown if the associated camel objects could not be deactivated within the specified timeout.

## Consumer Actors

For objects to receive messages, they must mixin the @extref[Consumer](github:akka-camel/src/main/scala/akka/camel/Consumer.scala)
trait. For example, the following actor class (Consumer1) implements the
endpointUri method, which is declared in the Consumer trait, in order to receive
messages from the `file:data/input/actor` Camel endpoint.

@@snip [Consumers.scala]($code$/scala/docs/camel/Consumers.scala) { #Consumer1 }

Whenever a file is put into the data/input/actor directory, its content is
picked up by the Camel [file component](http://camel.apache.org/file2.html) and sent as message to the
actor. Messages consumed by actors from Camel endpoints are of type
@extref[CamelMessage](github:akka-camel/src/main/scala/akka/camel/CamelMessage.scala). These are immutable representations of Camel messages.

Here's another example that sets the endpointUri to
`jetty:http://localhost:8877/camel/default`. It causes Camel's Jetty
component to start an embedded [Jetty](http://www.eclipse.org/jetty/) server, accepting HTTP connections
from localhost on port 8877.

@@snip [Consumers.scala]($code$/scala/docs/camel/Consumers.scala) { #Consumer2 }

After starting the actor, clients can send messages to that actor by POSTing to
`http://localhost:8877/camel/default`. The actor sends a response by using the
sender *!* method. For returning a message body and headers to the HTTP
client the response type should be @extref[CamelMessage](github:akka-camel/src/main/scala/akka/camel/CamelMessage.scala). For any other response type, a
new CamelMessage object is created by akka-camel with the actor response as message
body.

<a id="camel-acknowledgements"></a>
### Delivery acknowledgements

With in-out message exchanges, clients usually know that a message exchange is
done when they receive a reply from a consumer actor. The reply message can be a
CamelMessage (or any object which is then internally converted to a CamelMessage) on
success, and a Failure message on failure.

With in-only message exchanges, by default, an exchange is done when a message
is added to the consumer actor's mailbox. Any failure or exception that occurs
during processing of that message by the consumer actor cannot be reported back
to the endpoint in this case. To allow consumer actors to positively or
negatively acknowledge the receipt of a message from an in-only message
exchange, they need to override the `autoAck` method to return false.
In this case, consumer actors must reply either with a
special akka.camel.Ack message (positive acknowledgement) or a akka.actor.Status.Failure (negative
acknowledgement).

@@snip [Consumers.scala]($code$/scala/docs/camel/Consumers.scala) { #Consumer3 }

<a id="camel-timeout"></a>
### Consumer timeout

Camel Exchanges (and their corresponding endpoints) that support two-way communications need to wait for a response from
an actor before returning it to the initiating client.
For some endpoint types, timeout values can be defined in an endpoint-specific
way which is described in the documentation of the individual Camel
components. Another option is to configure timeouts on the level of consumer actors.

Two-way communications between a Camel endpoint and an actor are
initiated by sending the request message to the actor with the @extref[ask](github:akka-actor/src/main/scala/akka/pattern/AskSupport.scala) pattern
and the actor replies to the endpoint when the response is ready. The ask request to the actor can timeout, which will
result in the [Exchange](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/Exchange.java) failing with a TimeoutException set on the failure of the [Exchange](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/Exchange.java).
The timeout on the consumer actor can be overridden with the `replyTimeout`, as shown below.

@@snip [Consumers.scala]($code$/scala/docs/camel/Consumers.scala) { #Consumer4 }

## Producer Actors

For sending messages to Camel endpoints, actors need to mixin the @extref[Producer](github:akka-camel/src/main/scala/akka/camel/Producer.scala) trait and implement the endpointUri method.

@@snip [Producers.scala]($code$/scala/docs/camel/Producers.scala) { #Producer1 }

Producer1 inherits a default implementation of the receive method from the
Producer trait. To customize a producer actor's default behavior you must override the @extref[Producer](github:akka-camel/src/main/scala/akka/camel/Producer.scala).transformResponse and
@extref[Producer](github:akka-camel/src/main/scala/akka/camel/Producer.scala).transformOutgoingMessage methods. This is explained later in more detail.
Producer Actors cannot override the default @extref[Producer](github:akka-camel/src/main/scala/akka/camel/Producer.scala).receive method.

Any message sent to a @extref[Producer](github:akka-camel/src/main/scala/akka/camel/Producer.scala) actor will be sent to
the associated Camel endpoint, in the above example to
`http://localhost:8080/news`. The @extref[Producer](github:akka-camel/src/main/scala/akka/camel/Producer.scala) always sends messages asynchronously. Response messages (if supported by the
configured endpoint) will, by default, be returned to the original sender. The
following example uses the ask pattern to send a message to a
Producer actor and waits for a response.

@@snip [Producers.scala]($code$/scala/docs/camel/Producers.scala) { #AskProducer }

The future contains the response CamelMessage, or an `AkkaCamelException` when an error occurred, which contains the headers of the response.

<a id="camel-custom-processing"></a>
### Custom Processing

Instead of replying to the initial sender, producer actors can implement custom
response processing by overriding the routeResponse method. In the following example, the response
message is forwarded to a target actor instead of being replied to the original
sender.

@@snip [Producers.scala]($code$/scala/docs/camel/Producers.scala) { #RouteResponse }

Before producing messages to endpoints, producer actors can pre-process them by
overriding the @extref[Producer](github:akka-camel/src/main/scala/akka/camel/Producer.scala).transformOutgoingMessage method.

@@snip [Producers.scala]($code$/scala/docs/camel/Producers.scala) { #TransformOutgoingMessage }

### Producer configuration options

The interaction of producer actors with Camel endpoints can be configured to be
one-way or two-way (by initiating in-only or in-out message exchanges,
respectively). By default, the producer initiates an in-out message exchange
with the endpoint. For initiating an in-only exchange, producer actors have to override the oneway method to return true.

@@snip [Producers.scala]($code$/scala/docs/camel/Producers.scala) { #Oneway }

### Message correlation

To correlate request with response messages, applications can set the
*Message.MessageExchangeId* message header.

@@snip [Producers.scala]($code$/scala/docs/camel/Producers.scala) { #Correlate }

### ProducerTemplate

The @extref[Producer](github:akka-camel/src/main/scala/akka/camel/Producer.scala) trait is a very
convenient way for actors to produce messages to Camel endpoints. Actors may also use a Camel 
`ProducerTemplate` for producing messages to endpoints.

@@snip [Producers.scala]($code$/scala/docs/camel/Producers.scala) { #ProducerTemplate }

For initiating a two-way message exchange, one of the
`ProducerTemplate.request*` methods must be used.

@@snip [Producers.scala]($code$/scala/docs/camel/Producers.scala) { #RequestProducerTemplate }

<a id="camel-asynchronous-routing"></a>
## Asynchronous routing

In-out message exchanges between endpoints and actors are
designed to be asynchronous. This is the case for both, consumer and producer
actors.

 * A consumer endpoint sends request messages to its consumer actor using the `!`
(tell) operator and the actor returns responses with `sender !` once they are
ready.
 * A producer actor sends request messages to its endpoint using Camel's
asynchronous routing engine. Asynchronous responses are wrapped and added to the
producer actor's mailbox for later processing. By default, response messages are
returned to the initial sender but this can be overridden by Producer
implementations (see also description of the `routeResponse` method
in [Custom Processing](#camel-custom-processing)).

However, asynchronous two-way message exchanges, without allocating a thread for
the full duration of exchange, cannot be generically supported by Camel's
asynchronous routing engine alone. This must be supported by the individual
Camel components (from which endpoints are created) as well. They must be
able to suspend any work started for request processing (thereby freeing threads
to do other work) and resume processing when the response is ready. This is
currently the case for a [subset of components](http://camel.apache.org/asynchronous-routing-engine.html) 
such as the Jetty component.
All other Camel components can still be used, of course, but they will cause
allocation of a thread for the duration of an in-out message exchange. There's
also [Examples](#camel-examples) that implements both, an asynchronous
consumer and an asynchronous producer, with the jetty component.

If the used Camel component is blocking it might be necessary to use a separate
@ref:[dispatcher](dispatchers.md) for the producer. The Camel processor is
invoked by a child actor of the producer and the dispatcher can be defined in
the deployment section of the configuration. For example, if your producer actor
has path `/user/integration/output` the dispatcher of the child actor can be
defined with:

```
akka.actor.deployment {
  /integration/output/* {
    dispatcher = my-dispatcher
  }
}
```

## Custom Camel routes

In all the examples so far, routes to consumer actors have been automatically
constructed by akka-camel, when the actor was started. Although the default
route construction templates, used by akka-camel internally, are sufficient for
most use cases, some applications may require more specialized routes to actors.
The akka-camel module provides two mechanisms for customizing routes to actors,
which will be explained in this section. These are:

 * Usage of [camel components](#camel-components-2) to access actors.
Any Camel route can use these components to access Akka actors.
 * [Intercepting route construction](#camel-intercepting-route-construction) to actors.
This option gives you the ability to change routes that have already been added to Camel.
Consumer actors have a hook into the route definition process which can be used to change the route.

<a id="camel-components-2"></a>
### Akka Camel components

Akka actors can be accessed from Camel routes using the actor Camel component. This component can be used to
access any Akka actor (not only consumer actors) from Camel routes, as described in the following sections.

<a id="access-to-actors"></a>
### Access to actors

To access actors from custom Camel routes, the actor Camel
component should be used. It fully supports Camel's [asynchronous routing
engine](http://camel.apache.org/asynchronous-routing-engine.html).

This component accepts the following endpoint URI format:

 * `[<actor-path>]?<options>`

where `<actor-path>` is the `ActorPath` to the actor. The `<options>` are
name-value pairs separated by `&` (i.e. `name1=value1&name2=value2&...`).

#### URI options

The following URI options are supported:

|Name         | Type     | Default | Description                                                                                                                                                                                                                                                         |
|-------------|----------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|replyTimeout | Duration | false | The reply timeout, specified in the same way that you use the duration in akka, for instance `10 seconds` except that in the url it is handy to use a + between the amount and the unit, like for example `200+millis` See also [Consumer timeout](#camel-timeout).|
|autoAck      | Boolean  | true  | If set to true, in-only message exchanges are auto-acknowledged when the message is added to the actor's mailbox. If set to false, actors must acknowledge the receipt of the message.  See also [Delivery acknowledgements](#camel-acknowledgements).              |

Here's an actor endpoint URI example containing an actor path:

```
akka://some-system/user/myconsumer?autoAck=false&replyTimeout=100+millis
```

In the following example, a custom route to an actor is created, using the
actor's path. the Akka camel package contains an implicit `toActorRouteDefinition` that allows for a route to
reference an `ActorRef` directly as shown in the below example, The route starts from a [Jetty](http://www.eclipse.org/jetty/) endpoint and
ends at the target actor.

@@snip [CustomRoute.scala]($code$/scala/docs/camel/CustomRoute.scala) { #CustomRoute }

When a message is received on the jetty endpoint, it is routed to the Responder actor, which in return replies back to the client of
the HTTP request.

<a id="camel-intercepting-route-construction"></a>
### Intercepting route construction

The previous section, [camel components](#camel-components-2), explained how to setup a route to an actor manually.
It was the application's responsibility to define the route and add it to the current CamelContext.
This section explains a more convenient way to define custom routes: akka-camel is still setting up the routes to consumer actors (and adds these routes to the current CamelContext) but applications can define extensions to these routes.
Extensions can be defined with Camel's [Java DSL](http://camel.apache.org/dsl.html) or [Scala DSL](http://camel.apache.org/scala-dsl.html).
For example, an extension could be a custom error handler that redelivers messages from an endpoint to an actor's bounded mailbox when the mailbox was full.

The following examples demonstrate how to extend a route to a consumer actor for
handling exceptions thrown by that actor.

@@snip [CustomRoute.scala]($code$/scala/docs/camel/CustomRoute.scala) { #ErrorThrowingConsumer }

The above ErrorThrowingConsumer sends the Failure back to the sender in preRestart
because the Exception that is thrown in the actor would
otherwise just crash the actor, by default the actor would be restarted, and the response would never reach the client of the Consumer.

The akka-camel module creates a RouteDefinition instance by calling
from(endpointUri) on a Camel RouteBuilder (where endpointUri is the endpoint URI
of the consumer actor) and passes that instance as argument to the route
definition handler *). The route definition handler then extends the route and
returns a ProcessorDefinition (in the above example, the ProcessorDefinition
returned by the end method. See the [org.apache.camel.model](https://svn.apache.org/repos/asf/camel/tags/camel-2.8.0/camel-core/src/main/java/org/apache/camel/model/) package for
details). After executing the route definition handler, akka-camel finally calls
a to(targetActorUri) on the returned ProcessorDefinition to complete the
route to the consumer actor (where targetActorUri is the actor component URI as described in [Access to actors](#access-to-actors)).
If the actor cannot be found, a *ActorNotRegisteredException* is thrown.

*) Before passing the RouteDefinition instance to the route definition handler,
akka-camel may make some further modifications to it.

<a id="camel-examples"></a>
## Examples

The sample named @extref[Akka Camel Samples with Scala](ecs:akka-samples-camel-scala) (@extref[source code](samples:akka-sample-camel-scala))
contains 3 samples:

>
 * Asynchronous routing and transformation - This example demonstrates how to implement consumer and
producer actors that support [Asynchronous routing](#camel-asynchronous-routing) with their Camel endpoints.
 * Custom Camel route - Demonstrates the combined usage of a `Producer` and a
`Consumer` actor as well as the inclusion of a custom Camel route.
 * Quartz Scheduler Example - Showing how simple is to implement a cron-style scheduler by
using the Camel Quartz component

## Configuration

There are several configuration properties for the Camel module, please refer
to the @ref:[reference configuration](general/configuration.md#config-akka-camel).

## Additional Resources

For an introduction to akka-camel 2, see also the Peter Gabryanczyk's talk [Migrating akka-camel module to Akka 2.x](http://skillsmatter.com/podcast/scala/akka-2-x).

For an introduction to akka-camel 1, see also the [Appendix E - Akka and Camel](http://www.manning.com/ibsen/appEsample.pdf)
(pdf) of the book [Camel in Action](http://www.manning.com/ibsen/).

Other, more advanced external articles (for version 1) are:

 * [Akka Consumer Actors: New Features and Best Practices](http://krasserm.blogspot.com/2011/02/akka-consumer-actors-new-features-and.html)
 * [Akka Producer Actors: New Features and Best Practices](http://krasserm.blogspot.com/2011/02/akka-producer-actor-new-features-and.html)