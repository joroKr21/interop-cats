---
id: index
title: "ZIO 2.x Interoperation with Cats 3.x"
sidebar_label: "ZIO 2.x Interop Cats 3.x"
---

## Installation

```sbt
libraryDependencies += "dev.zio" %% "zio-interop-cats" % "<latest-version>"
```

## `ZIO` Cats Effect 3 instances

**ZIO** integrates with Typelevel libraries by providing an instance of `Concurrent`, `Temporal` and `Async` for `Task`
as required, for instance, by `fs2`, `doobie` and `http4s`.

For convenience, the ZIO library defines an alias as follows:

```scala
type Task[A] = ZIO[Any, Throwable, A]
```

Therefore, we provide Cats Effect instances based on this specific datatype.

## `Concurrent`

In order to get a `Concurrent[Task]` or `Concurrent[RIO[R, *]]` (note `*` is kind-projector notation) we need to import `zio.interop.catz._`:

```scala
import cats.effect._
import zio._
import zio.interop.catz._

def ceConcurrentForTaskExample = {
  val F: cats.effect.Concurrent[Task] = implicitly
  F.racePair(F.unit, F.unit)
}
```

## `Temporal`

```scala
import cats.effect._
import zio._
import zio.interop.catz._

def ceTemporal = {
  val F: cats.effect.Temporal[Task] = implicitly
  F.sleep(1.second) *> F.unit
}
```

## `Async`

```scala
import cats.effect._
import zio._
import zio.interop.catz._

def ceAsync = {
  val F: cats.effect.Async[Task] = implicitly
  F.racePair(F.unit, F.sleep(1.second) *> F.unit)
}
```

## Other typeclasses

There are many other typeclasses and useful conversions that this library provides implementations for:
* See `zio/interop/cats.scala` file to see all available typeclass implementations for the Cats Effect 3 typeclasses
* See `zio/stream/interop/cats.scala` for ZStream typeclass implementations
* See `zio/stream/interop/FS2StreamSyntax.scala` for FS2 <-> ZStream conversions


### cats-core

If you only need instances for `cats-core` typeclasses, not `cats-effect` import `zio.interop.catz.core._`:

```scala
import zio.interop.catz.core._
```

Note that this library only has an `Optional` dependency on cats-effect – if you or your libraries don't depend on it, this library will not add it to the classpath.

### Example

The following example shows how to use ZIO with Doobie (a library for JDBC access) and FS2 (a streaming library), which both rely on Cats Effect instances (`cats.effect.Async` and `cats.effect.Temporal`):

```scala
import zio.{durationInt as _, *}
import zio.interop.catz.*
import doobie.*
import doobie.implicits.*
import scala.concurrent.duration.*

object Example extends ZIOAppDefault:
  val run = {
    val xa: Transactor[Task] =
      Transactor.fromDriverManager[Task]("org.h2.Driver", "jdbc:h2:mem:test;DB_CLOSE_DELAY=-1", "user", "", None)

    sql"SELECT 42"
      .query[Int]
      .stream
      .transact(xa)
      .delayBy(1.second)
      .evalTap(i => Console.printLine(i))
      .compile
      .drain
      .exitCode
  }
```
