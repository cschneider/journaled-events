OSGi Journaled Events
=====================

The goal of this effort is to provide an API (possibly as spec) and backends for journaled streams of events. These extend the publish/subscribe model with means to start consume from an point in the stored event stream history.

## What

The [OSGi Compendium specification](https://osgi.org/javadoc/osgi.cmpn/7.0.0/) defines the APIs, in the [org.osgi.service.event.\*](https://osgi.org/javadoc/osgi.cmpn/7.0.0/org/osgi/service/event/package-frame.html) packages, to support the distribution of events via the Publish/Subscribe messaging model.

The Publish/Subscribe messaging model covers the distribution of events form a publisher to multiple subscribers. The published events are consumed by all the registered consumers at the time the events are produced.

A key feature of the Publish/Subscribe messaging model is to avoid coupling the publishers and subscribers.
This decoupling is achieved by sending events to topics and consuming events from topics rather than individual producers or subscribers.

A key drawback of the Publish/Subscribe messaging model resides in the fact that subscribers only consume the events produced after their subscription.

As long as no guarantee of consumption is required, the Publisher/Subscriber model is effective and allow to nicely communicate events across the bundles/components boundaries.

However, use cases requiring a guarantee of consumption (e.g. at-least-once, at-most-once, exactly-once) can't be supported by a Publish/Subscribe model only.
Offering guarantee of consumption requires giving each subscriber access to events produced before their subscription (historical events).
The history of events would typically be stored in queues for later access.

This proposal aims at extending the OSGi specification in order to allow consumers to access historical (journaled) events towards enforcing guarantees of consumption.

## Why

No OSGi specification or API exists to support a Publish/Subscriber event distribution with guarantee of consumption.
However, use case requiring those guarantees exist.

In practice, projects implement their own ad-hoc queueing mechanism to support a form of journaled events.
Those queueing mechanism are often brittle, require duplicating code, may or may not rely on proven message queueing infrastructure.

By adding a specification and API to OSGi for journaled events, we would offer a standardise interface to this common feature.
The implementations of this API will be swappable and sharable among projects towards avoiding ad-hoc solutions to a general use case.

## Goals

* Provide traditional publish / subscribe semantics
* Allow consuming a stream from any point in the history (given it is not yet evicted)

## Non Goals

* We explicitly do not cover the extreme scaling of Apache Karaf. So no sharding support in the API (like partitions).

## How

Journaled events must keep the Publish/Subscribe key feature, decoupling the event publishers and subscribers.
However, journaled events must give consumers access to historical data.

The [Apache Kafka](https://kafka.apache.org/) publish/subscribe API provides such decoupled journaled distribution of messages.
Apache Kafka persists messages reliably (according to a configurable retention policy) and indexes each message with monotonically increasing offsets.
In a nutshell, those offsets specify the ordering of messages as they are produced.
The offsets are also used by consumers to define the entry point for consuming messages, potentially starting from a messages associated to an offset produced before the consumer started.
Apache Kafka API allows the consumers to maintain their own consumed offsets.
This model is interesting as it offers the possibility to provide [exactly-once](https://kafka.apache.org/21/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html) guarantee of processing when the consumer can persist the consumed offset atomically together with the side effect of processing the message.

OSGi journaled events should leverage the Apache Kafka messaging model but in a simplified way.
Indeed, Apache Kafka focuses on distributing messages reliably and at high throughput across processes were OSGi journaled events could limit to distributing journal events in a single process.
A simple journaled event model lowers the bar for providing implementations specialised to a context.

The most natural implementation of the journaled event API would be on Apache Kafka, however we foresee that an in-memory implementation and implementation on top of backend supporting sequence natively will be useful as well.
This should be the case for any RDB, for some document based backends (e.g. Collection on MongoDB).

## First sketch of an API

```java
    interface JournaledMessaging {
        void send(String topic, Message message);
        Closeable subscribe(String topic, Position pos, Consumer<Message> callback);
        Message newMessage(byte[] payload, Map<String, String> props);
        Position positionFromString(String out);
        String positionToString(Position in);
    }

    interface Message {
        byte[] getPayload();
        Position getPosition();
        Map<String, String> getProperties();
    }

    interface Seek {
        Seek earliest = new Seek() {};
        Seek latest = new Seek() {};
    }

    interface Position extends Seek {};
```
