---
layout: post
title:  "Consulting toolkit patterns: Service Function"
date:   2016-02-28 12:20:00 -0800
categories: scala
---

This is the first post in a series in which I will describe some patterns I'm developing into a consulting toolkit. It is very likely that I reinvent the wheel at times as I develop ideas. This is how I like to learn: put blinders on and try to invent things before looking externally to see what has already been invented. I expect to revisit all patterns once they are fairly baked and try to resolve them to existing concepts - at which time I'll change names to reflect the "standard" patterns. If I come up with a novel pattern, then woohoo for me!

## Definition: Service

A __Service__ is as an object that responds to requests from clients. If you're like me and you come from an imperative background, you might immediately imagine that a __Service__ implements an interface. Something like:

```scala
import java.time.Instant

case class Foo(id: String, name: String, lastUpdated: Instant)

/** Service for interacting with Foo data values */
trait FooRepository {
  def create(name: String): Foo
  def read(id: String): Option[Foo]
  def update(id: String, updater: Foo => Foo): Foo
  def delete(id: String): Unit
  def all: Iterable[Foo]
  def fetch(p: Foo => Boolean): Iterable[Foo] = all.filter(p)
}
```

## Definition: Service Function

A __Service Function__ (as I'm defining here) is a _function_, so it will not implement an interface of methods. Instead, it will implement a single `apply` method. The `FooRepository` service above written as a __Service Function__ might look like:

```scala
trait FooRepository {
  def apply[Response](request: FooRequest[Response]): Response
}
```

Where a `FooRequest` is an [algebraic data type][adt] (ADT) parameterized by a response type:

```scala
sealed trait FooRequest[Response]
object FooRequest {
  case object All extends FooRequest[Iterable[Foo]]
  case class Fetch(p: Foo => Boolean) extends FooRequest[Iterable[Foo]]
  case class Create(name: String) extends FooRequest[Foo]
  case class Read(id: String) extends FooRequest[Option[Foo]]
  case class Update(id: String, updater: Foo => Foo) extends FooRequest[Foo]
  case class Delete(id: String) extends FooRequest[Unit]
}
```

Notice that each concrete request type provides a `Response` type. For example, `Read` has a response type of `Option[Foo]`. Requiring that the response type be specified by each concrete request type enables our service function to be _polymorphic_. In other words, the return type of our service function is statically typed to be the response type of the request object.

Client code would use the `FooRepository` service function like so:

```scala
class BarService(fooRepository: FooRepository) {
  def updateAllFoos(): Unit = 
    fooRepository(FooRequest.All) foreach { foo =>
      fooRepository(FooRequest.Update(foo.id, _.lastUpdated = Instant.now))
    }
}
```

## Motivation

What are the rewards for writing services as __Service Functions__? 

As written above, the only real difference between the service defined as a trait with abstract methods and the service defined as a function is that the service's interface is expressed by an ADT in the latter. Having the ADT separate from the service introduces flexibility (you can use the same ADT in other places - e.g. as messages to actors). 

ADTs in place of method signatures help to orient design around the domain models (think [domain driven design][ddd]).

If we can limit our design to consist of data models, ADTs, and functions that operate on them, then we naturally fall into functional programming (at least this is how I'm seeing it currently).

## Adding Response Wrappers

One problem with our above example is that there is no handling of errors. You can imagine a recoverable exception being thrown in the underlying persistence layer. We want to be good functional programmers and not allow uncaught recoverable exceptions in our code. What if we wrapped all of our service responses in a `scala.util.Try`?

```scala
import scala.util.Try

trait FooRepository {
  def apply[Response](request: FooRequest[Response]): Try[Response]
}
```

And what if we want an async version of our `FooRepository`? Something like:

```scala
import scala.concurrent.Future

trait AsyncFooRepository {
  def apply[Response](request: FooRequest[Response]): Future[Response]
}
```

You can see that a pattern is emerging - we have two repositories that are different only in the containing type wrapping responses. We can extract the wrapping response type as a parameter to a generic `FooRepository`:

```scala
trait FooRepository[M[_]] {
  def apply[Response](request: FooRequest[Response]): M[Response]
}

trait SyncFooRepository extends FooRepository[Try]
trait AsyncFooRepository extends FooRepository[Future]
```

## Generic Service Function

For my consulting toolkit (code to be opened up once more baked), I want a generic `Service` trait that I can use for all service objects. This trait should promote consistent style and not restrict design. A generic `Service` function:

```scala
/** A Service function that returns responses contained by another type.
  *
  * @tparam M[_] the higher-kinded type that will wrap all responses. Commonly will be a Future or Try
  * @tparam Request[_] the request ADT
  */
trait Service[M[_], Request[_]] {
  /** @tparam Response the response type that will be returned within an M. Implied by the request argument's type.
    * @param request a request object that is typed by its response type
    */
  def apply[Response](request: Request[Response]): M[Response]
}
```

__Note__: the `M[_]` and `Request[_]` type parameters are what is known as a "higher kinded" types and will cause a compiler error telling you that you must import `scala.language.higherkinds`. Instead of following the compiler error's suggestion, I recommend setting up your build with the scalac compiler flags per [tpolecat's recommendation][tpolecat-scalac-flags]. This will also cause all compiler warnings to be considered errors - which is important for exhaustive pattern matching (which you will do in `Service` implementations).

We can provide `Service` types that have the wrapper type already set:

```scala
import scala.concurrent.Future
import scala.util.Try

trait AsyncService[Request[_]] extends Service[Future, Request]
trait SyncService[Request[_]] extends Service[Try, Request]
```

Now we can have:

```scala
trait SyncFooRepository extends SyncService[FooRequest]
trait AsyncFooRepository extends AsyncService[FooRequest]
```

## Mapping an AsyncService to a SyncService

I frequently work on non-blocking web applications. Often times, we need CLI that use the services defined in our production web application. It can be annoying to work with async services in a CLI. It would be great if I could map an `AsyncService` to a `SyncService` and just use the synchronous version. Here's how we can do that:

```scala
trait SyncFooRepository extends SyncService[FooRequest]
trait AsyncFooRepository extends AsyncService[FooRequest]

object SyncService {
  import scala.concurrent.Await
  import scala.concurrent.duration.Duration

  /** Converts an AsyncService to a SyncService by blocking on responses.
    *
    * @tparam Request[_] the request ADT
    * @param asyncService the service to convert to blocking
    * @param timeout the timeout to use when blocking on responses
    */
  def apply[Request[_]](asyncService: AsyncService[Request], timeout: Duration = Duration.Inf): SyncService[Request] = {
    new SyncService[Request] {
      val mapped = asyncService map (resp => Try(Await.result(resp, timeout)))
      override def apply[Response](request: Request[Response]): Try[Response] = mapped(request)
    }
  }
}
```

And now we can do this:

```scala
object CLI extends App {
  // We have only one FooRepository implementation: AsyncFooRepositoryImpl
  // We can create a synchronous version easily:
  val fooRepo: AsyncService[FooRequest] = SyncService(AsyncFooRepositoryImpl)
  
  // do stuff with the repository synchronously...
}
```

[scala-world-youtube]: https://www.youtube.com/channel/UCc0j7uOItUDh7vEvPb-TeCg
[debasish-domain-modeling]: https://www.youtube.com/watch?v=U0Rk9Knq8Vk
[adt]: http://noelwelsh.com/programming/2015/06/02/everything-about-sealed/#algebraic-data-types
[ddd]: https://en.wikipedia.org/wiki/Domain-driven_design
[tpolecat-scalac-flags]: https://tpolecat.github.io/2014/04/11/scalac-flags.html
