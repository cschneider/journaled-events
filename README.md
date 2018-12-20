OSGi journaled-events
=====================

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

This proposal aims at extending the OSGi specification in order to allow consumers to access historical events towards enforcing guarantees of consumption.

## Why

No OSGi specification or API exists to support a Publish/Subscriber event distribution with guarantee of consumption.
However, use case requiring those guarantees exist.

In practice, projects implement their own ad-hoc queueing mechanism to support a form of journaled events.
Those queueing mechanism are often brittle, require duplicating code, may or may not rely on proven message queueing infrastructure.

By adding a specification and API to OSGi for journaled events, we would offer a standardise interface to this common feature.
The implementations of this API will be swappable and sharable among projects towards avoiding ad-hoc solutions to a general use case.
