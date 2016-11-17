---
layout: post
title:  "Event Sourcing with Akka Persistence and Apache Thrift"
date:   2016-11-17 12:20:00 -0800
categories: scala
---

The most difficult barrier I experienced in getting set up with [Akka Persistence][akka-persistence] for production [Event Sourcing][event-sourceing] set up for production was figuring out a proper serialization strategy. The [Akka Persistence docs][akka-doc-persistence] mention the importance of not using the default Java serialization in production due to the fact that schema migrations become impossible. Instead, it is recommended to use something like [Google's Protocol Buffers][protobufs], [Apache Thrift][thrift], or [Apache Avro][avro] each of which provide mechanisms for managing binary serialization that is robust to (most kinds of) schema changes.

## The Event Sourcing Promise of Simplicity

From an application architecture perspective, event sourcing promises simplicity and scalability. Events are write-only, and domain models' state are a pure function of `Seq[Event] => ModelState`.

## The Serialization Tradeoff

When getting started with the default Java serialization, you can focus on your models.
Introducing a "serialization" strategy using a library such as ones mentioned above means the ground truth for models is defined in another language. You then have to either use code-generated representations of those models, or hand write a mapping layer that maps the code generarted representatons to the actual domain models used in your application. I feel strongly the latter strategy is the correct strategy for several reasons.

[protobufs]: https://developers.google.com/protocol-buffers/
[thrift]: https://thrift.apache.org/
[avro]: https://avro.apache.org
[akka-doc-persistence]: http://doc.akka.io/docs/akka/2.4/scala/persistence.html
[akka-doc-event-adapters]: http://doc.akka.io/docs/akka/2.4/scala/persistence.html#Event_Adapters
