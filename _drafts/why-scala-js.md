---
layout: post
title:  "Why Scala.js Rocks"
date:   2016-02-28 12:20:00 -0800
categories: scala
---

I've become convinced that [Scala.js][scala-js] with [scalajs-react][scalajs-react], and [scalacss][scalacss] is a superior way to do frontend development when the backend is written in Scala. "Superior to what" you may ask? Previously, I had tried [sbt-web][sbt-web] before deciding that plain [npm][npm] + [gulp][gulp] + [babel][babel] + [react][react] + [less][less] was the way to go. I wrote an [SBT plugin][sbt-plugins-nodejs] to facilitate calling out to `npm` in SBT builds. This seemed like the best approach because:

- We could use the latest and greatest libraries on npm
- We could use the latest and greatest nodejs build tooling
- It is the most "standard" way of building UIs which means more documentation and bigger communities for support

## But npm isn't perfect...

While delegating to `npm` allowed for great freedom, it proved very costly. Some costs included:

- **Coding in Javascript**. I know it is an obvious argument (and tiresome to read) to call Javascript inferior to Scala. Instead of comparing languages, I'll focus on language switching costs. There is the obvious cost of having to keep in your head syntactical differences between Scala and Javascript. Perhaps worse is coding style differences. We wanted to code in an "immutable" style in Javascript, which led us to adopt [Immutable.js][immutable-js] which provides Scala collections-like API. However, there is significant cognitive overhead learning a library like Immutable.js which is different enough from the scala collections...
- **Developing and maintaining `Gulpfile`s**. It is hard enough not to forget how to work with SBT between build tweakings... having a second build tool in another language running in another runtime...
- **Depending on Node.js for builds**. Not only is this a configuration issue (must have correct version of `npm` installed on machines), but it slows down builds (at least it does in our configurations). We also have to deal with node_module caching issues from time to time.
- **Keeping Server and Client models compatible**. This is huge. If someone changes a model on the server that we serialize to JSON for client consumption, there is no compile-time check that we haven't broken the contract with the UI. Testing for breakages will require spinning up an environment and running integration tests. With Scala.js you can use the same models server-side and client-side, serializing and deserializing using the same [JSON library][upickle].

## "Scala.js sounds like GWT"

Many (including me) have bad memories of developing GWT applications. Scala.js is _not_ GWT. Scala.js is much more like [Typescript][typescript] - a language that compiles to Javascript. The code you write will look very similar to Javascript.

## When Scala.js Becomes "worth it"

- When the client is > 1000 LOC ?

[scala-js]: http://www.scala-js.org/
[sbt-web]: https://github.com/sbt/sbt-web
[scalajs-react]: https://github.com/japgolly/scalajs-react
[scalacss]: http://japgolly.github.io/scalacss/book/
[npm]: https://www.npmjs.com/
[gulp]: http://gulpjs.com/
[babel]: https://babeljs.io/
[react]: https://facebook.github.io/react/
[sbt-plugins-nodejs]: https://github.com/allenai/sbt-plugins/blob/master/src/main/scala/org/allenai/plugins/NodeJsPlugin.scala
[less]: http://lesscss.org/
[immutable-js]: https://facebook.github.io/immutable-js/
[upickle]: http://www.lihaoyi.com/upickle-pprint/upickle/
[typescript]: http://www.typescriptlang.org/
