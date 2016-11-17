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

## The Serialization Library Selection Journey

Early analysis of the three libraries mentioned resulted in my prioritizing their consideration in the following order:

| Rank   | Library      | Reason for Ranking |
| :----: | ------------ | ------------------ |
| 1      | [Avro][avro] | Potentially no need for code generation since schemas are serialized with the objects and can be generated at compile time via [avro4s][avro4s] library. There was a simple [blog post][avro4s-akka-blog] showing how to get set up with Akka Persistence and Avro (without code generation) which looked to be exactly what I wanted. |
| 2      | [Protobuf][protobufs] | Since Akka itself uses protobufs, and Google has behind it for a long time, seemed like the "safe" bet if we end up having to fall back to code generation. |
| 3      | [Thrift][thrift] | Seemed the least visible project - at least from my early analysis. |

TODO

### Avro: a failed promise

- Had [macro compiler errors][avro4s-issue-macro] that couldn't figure out (despite forking the repo and attempting to recreate issue via unit tests (could not)). In the process of debugging, contributed a [cleanup PR][avro4s-pr] to eliminate compiler warnings in the project.
- Even if I had gotten it to work, I would have certainly suffered more pains due to [poor support of ADTs][avro4s-issue-adt] (sealed traits).

The macro compiler errors were the show stopper for me. This meant that I was going to now consider code generation - either with Google Protobufs or with Apache Thrift.

### Google Protobufs: Surprise lack of good tooling

I was surprised to find that there wasn't great tooling (SBT) support for Google Protobufs. From my searching, the official [SBT project][sbt-protobuf] seemed to be it, and was really low level. I was discouraged and figured before moving forward I should check the state of things for Thrift.

### Thrift: Scrooge for the win!

I stumbled upon Twitter's open source Scrooge project ([repo][scrooge-repo], [docs][scrooge-site]). I started off reading through the [docs][scrooge-site], which seemed pretty sparse and outdated (example setup referenced very old library versions). This scared me a bit as I wondered if the project had been abandoned. But then I checked out the project's [repo][scrooge-repo] and was delighted to see the project was still actively being developed. Also, to my great delight, I found that scrooge has great support for ADTs using Thrift's [union][scrooge-union] definition feature. This was potentially a huge win for me because I rely heavily on ADTs when developing Scala projects. Yay for type safety and exhaustive pattern matching!

So, I dove in with Thrift and created a new SBT subproject to house my `.thrift` files. The team at Twitter did a fantastic job with Scrooge, me thinks. It was easy to set up - though I stopped looking at the docs and just read the source files since the docs were so old. Once set up was complete, I added a source dependency on the `thrift` subproject in my `server` subproject and went to work writing the mapping layer from the generated thrift struct models to my domain models. This was easy to do, and thanks to the generation of ADTs from `union` definitions mapping was typesafe and comprehensive via exhaustive pattern matching. TODO: show some example thrift source files and mapping code.

[protobufs]: https://developers.google.com/protocol-buffers/
[thrift]: https://thrift.apache.org/
[avro]: https://avro.apache.org
[avro4s-akka-blog]: http://giampaolotrapasso.com/akka-persistence-using-avro-serialization/
[avro4s]: https://github.com/sksamuel/avro4s
[avro4s-issue-macro]: https://github.com/sksamuel/avro4s/issues/76
[avro4s-issue-adt]: https://github.com/sksamuel/avro4s/issues/62
[avro4s-pr]: https://github.com/sksamuel/avro4s/pull/77
[akka-doc-persistence]: http://doc.akka.io/docs/akka/2.4/scala/persistence.html
[akka-doc-event-adapters]: http://doc.akka.io/docs/akka/2.4/scala/persistence.html#Event_Adapters
[sbt-protobuf]: https://github.com/sbt/sbt-protobuf
[scrooge-repo]: https://github.com/twitter/scrooge
[scrooge-site]: http://twitter.github.io/scrooge/
[scrooge-union]: http://twitter.github.io/scrooge/GeneratedCodeUsage.html#unions
