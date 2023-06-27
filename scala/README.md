# Scala Code Guide

## Table of contents

- [Lizards Scala Code Guide](#lizards-scala-code-guide)
  - [Table of contents](#table-of-contents)
  - [References](#references)
    - [Inspiration](#inspiration)
    - [Draft and planning](#draft-and-planning)
- [Safety](#safety)
  - [Don’t use type aliases for public contracts](#dont-use-type-aliases-for-public-contracts)
  - [Use the type system to prevent errors](#use-the-type-system-to-prevent-errors)
  - [Case Classes](#case-classes)
  - [Avoid Booleans as data structures in Scala](#avoid-booleans-as-data-structures-in-scala)
  - [Try to make data correct by construction](#try-to-make-data-correct-by-construction)
  - [Use `sealed trait` for union types](#use-sealed-trait-for-union-types)
    - [Enumerations](#enumerations)
  - [Aim for pure functions](#aim-for-pure-functions)
  - [Favour lazy effects over eager effects](#favour-lazy-effects-over-eager-effects)
  - [Resource acquisition](#resource-acquisition)
  - [Mutable state](#mutable-state)
  - [Convert from unsafe Java models to Scala case class](#convert-from-unsafe-java-models-to-scala-case-class)
  - [Process termination](#process-termination)
- [Design goals](#design-goals)
  - [Make first-class test implementations](#make-first-class-test-implementations)
    - [Benefits:](#benefits)
      - [Better reuse](#better-reuse)
      - [Less fragile tests](#less-fragile-tests)
      - [Improve the quality and readability of test assertions](#improve-the-quality-and-readability-of-test-assertions)
  - [Pull out pure functions into companion object](#pull-out-pure-functions-into-companion-object)
  - [Create application config and pass it in where required](#create-application-config-and-pass-it-in-where-required)
  - [Prefer configuration settings to live in the codebase](#prefer-configuration-settings-to-live-in-the-codebase)
  - [Single scheduler between Futures and Tasks](#single-scheduler-between-futures-and-tasks)
  - [Project structure consistency across our repos](#project-structure-consistency-across-our-repos)
    - [Package-by-feature](#package-by-feature)
    - [Package Objects](#package-objects)
  - [Different types of testing](#different-types-of-testing)
    - [Unit tests](#unit-tests)
    - [Integration tests](#integration-tests)
    - [End to end tests](#end-to-end-tests)
  - [Avoid mocks](#avoid-mocks)
    - [Define a Lean and precise function contract to avoid mocks](#define-a-lean-and-precise-function-contract-to-avoid-mocks)
    - [Define loosely coupled modules/units to avoid mocks](#define-loosely-coupled-modulesunits-to-avoid-mocks)
    - [Test IO with real resources instead of mocking](#test-io-with-real-resources-instead-of-mocking)
    - [See also: Make first-class test implementations](#see-also-make-first-class-test-implementations)
  - [Limit the usage of implicit parameters](#limit-the-usage-of-implicit-parameters)
    - [Implicit methods](#implicit-methods)
    - [Implicit parameters for dependency injection](#implicit-parameters-for-dependency-injection)
    - [Implicit classes for extension methods](#implicit-classes-for-extension-methods)
    - [Typeclasses](#typeclasses)
  - [Techniques for reducing unnecessary diffs](#techniques-for-reducing-unnecessary-diffs)
    - [Wrapper functions for the class / method under test](#wrapper-functions-for-the-class--method-under-test)
- [Zendesk](#zendesk)
  - [Don't use `zendesk/sbt-base-settings`](#dont-use-zendesksbt-base-settings)
  - [Continuously deploy to staging](#continuously-deploy-to-staging)
    - [Using Samson](#using-samson)
    - [Using Spinnaker](#using-spinnaker)
- [Scala Libraries](#scala-libraries)
  - [Justification for `Cats`](#justification-for-cats)
  - [Justification for `Monix`](#justification-for-monix)
  - [Don't use the twitter stack](#dont-use-the-twitter-stack)
  - [Use `===` rather than `==` for equality](#use--rather-than--for-equality)
    - [Operator `===`](#operator-)
    - [Test framework](#test-framework)
- [Hygiene](#hygiene)
  - [No casting `asInstanceOf`](#no-casting-asinstanceof)
  - [Use `if` for booleans, not a pattern match](#use-if-for-booleans-not-a-pattern-match)
  - [Keep dependencies up to date](#keep-dependencies-up-to-date)
  - [Use dots and brackets `foo.bar(1)` instead of `foo bar 1`.](#use-dots-and-brackets-foobar1-instead-of-foo-bar-1)
  - [Use `Try[T]` instead of exception](#use-tryt-instead-of-exception)
  - [Never use `Await.result` outside the main thread (or tests)](#never-use-awaitresult-outside-the-main-thread-or-tests)
  - [Thread creation (including as part of thread pools):](#thread-creation-including-as-part-of-thread-pools)
  - [Favour composition over inheritance](#favour-composition-over-inheritance)
  - [No default parameter values in src/main which are only for tests](#no-default-parameter-values-in-srcmain-which-are-only-for-tests)
  - [Exhastively pattern match on `Option`](#exhastively-pattern-match-on-option)
  - [No `Seq` - Use `List`, `Chain` or `Vector`](#no-seq---use-list-chain-or-vector)
  - [No null](#no-null)
    - [Initialise with Option instead of `null`](#initialise-with-option-instead-of-null)
    - [Never return `null` from methods](#never-return-null-from-methods)
  - [Use immutable data structures](#use-immutable-data-structures)
  - [Prefer `val` over `var` declarations](#prefer-val-over-var-declarations)
  - [Don't let serialization corrupt your core model just to match the shape of expected json input / output](#dont-let-serialization-corrupt-your-core-model-just-to-match-the-shape-of-expected-json-input--output)
  - [Don't put multiple classes in one file](#dont-put-multiple-classes-in-one-file)
  - [Delete merged / unrequired branches](#delete-merged--unrequired-branches)
  - [Wrap a Case Class if you want a free toString / equals / hashcode](#wrap-a-case-class-if-you-want-a-free-tostring--equals--hashcode)
  - [Values representing times must include the time unit in the name](#values-representing-times-must-include-the-time-unit-in-the-name)
- [Still Up for debate](#still-up-for-debate)
  - [Test coverage](#test-coverage)
  - [The only things that should be public are those you expect to be called](#the-only-things-that-should-be-public-are-those-you-expect-to-be-called)
  - [Don't auto manage splitting code lines](#dont-auto-manage-splitting-code-lines)
  - [Use of parentheses for zero argument methods](#use-of-parentheses-for-zero-argument-methods)

## References
### Inspiration
https://github.com/alexandru/scala-best-practices

### Draft and planning
https://docs.google.com/document/d/1tcHzpiRngeP3LAkWULVRTrm3eCWJ6Jpex5Ux6hijTGI/edit#

# Safety
>  Covers all the "don't do X, it's dangerous or bad"

## Don’t use type aliases for public contracts
We shouldn’t use public type aliases as they don’t provide any guarantees.  Feel free to use as an internal implementation detail.

**Bad**
```scala
object Kafka {
  type ClusterName: String
  def getClusterName(): ClusterName = ???
}
```
**Good**
```scala
final case class ClusterName(value: String)

object Kafka {
  def getClusterName(): ClusterName = ???
}

```

|Pros|Cons|
-|-
|Compiler safety||

<br/>

## Use the type system to prevent errors
```scala
def mean(list: List[Int]): Int // bad
def mean(list: List[Int]): Either[String, Int] // better
def mean(list: NonEmptyList[Int]): Int // best
```
|Pros|Cons|
-|-
|Can prevent errors| |
|More explicit data types| |

## Case Classes

**Case Classes** should be used for things which are _just_ data (pure values) and only contains methods that operate on the properties of the case class and the operations are pure functions with non-effectful operations (e.g. a db request is an effectful operation that is usually wrapped in Future/Task/IO, etc.).

- :white_check_mark: prefer to have behaviour that is a pure function operating on the properties of the case class;
  ```scala
  case class BinlogLocation(filename: String, position: Long) {
    override def toString: String = s"$filename:$position"
  }
  ```

- :no_entry_sign: avoid to have a case class holding impure values like db connections, e.g.
  ```scala
  case class BinlogLocation(connection: DbConnection) {
    def position(filename: String): Try[Long] = {
      val statement = connection.createStatement
      val rs = statement.executeQuery(s"SELECT position FROM table where file = ${filename}")
      
      Try(rs[0].getLong("position"))
    }
  }
  ```

- :no_entry_sign: avoid to have a case class with method that operates on impure dependencies like db, e.g.
  ```scala
  case class BinlogLocation(filename: String) {
    def position(db: DB): Try[Long] = {
      val connection = db.getConnection(...)
      val statement = connection.createStatement
      val rs = statement.executeQuery(s"SELECT position FROM table where file = ${filename}")
      
      Try(rs[0].getLong("position"))
    }
  }
  ```


## Avoid Booleans as data structures in Scala
For critical business logic, booleans can sometimes be harder to follow than an ADT with 2 choices.

For example, consider:
```
case class BinlogReplicatorState(
    recovering: Boolean,
    ...
)

val state = BinlogReplicatorState(false)
```

The `false` is pretty obscure. Naming the parameter helps a little, but it can still be hard to follow.

**Good**
Instead consider modelling the two states explicitly. For example
```sealed trait RecoveryState
case object Recovering extends RecoveryState
case object NormalExecution extends RecoveryState

case class BinlogReplicatorState(
    recovering: RecoveryState,
    ...
)
```

The definition now becomes a lot clearer.
```
val state = BinlogReplicatorState(NormalExecution)
```

It also provides an additional level of type safety against accidentally switching two unrelated booleans.

Too see in context, here is an [example commit which switches from boolean to ADT](https://github.com/zendesk/escape/pull/229/commits/94461296184b698014801414090aa6ee7fe92f83).

## Try to make data correct by construction

This an _excellent_ way to eliminate bugs, wherever it can be applied. It serves two purposes:

 - reduce the amount of care and skill your users need to use your code _correctly_
 - make illegal states impossible to represent, allowing you to leverage the type checker for state validation

A classic example is the NonEmptyList. If you ever perform a check to make sure a `List[T]` is not empty, you should encode the result as a `NonEmptyList[T]` so that the compiler can propagate this extra knowledge for you, rather than humans trying to reason that "this list won't be empty because we already checked it", or adding additional checks "just in case".

For a more complex example, say you have a `BinlogEvent`. It can either have some data, or be a metadata event used for checkpointing the current position. A simple representation might look like:

```
case class BinlogEvent(position: BinlogPosition, data: Option[BinlogRow], isCheckpoint: boolean)
```

There are multiple problems here:

 - The code which does checkpointing must be careful to only take action only when `isCheckpoint` is true, checkpointing a non-checkpoint event would be a bug.
 - Code dealing with data events must will have to explicitly deal with the possibility of `data` being `None`, which can be inconvenient and repetitive.

To solve this first problem, you could write:

```
case class BinlogEvent(checkpointPosition: Option[BinlogPosition], data: Option[BinlogRow])
```

(In general, where you have a boolean in your data model, you should consider whether you can combine this with the data it affects, producing a single `Option[T]` or `Either[A,B]`)

With this change any checkpointing code would find it impossible to checkpoint the position of an event which is not meant for checkpointing, which is great. But that doesn't solve the second problem above. For that there's still a better model - the event is _either_ data or a checkpoint, which we can cleanly represent as:

```
sealed trait BinlogEvent
case class DataEvent(data: BinlogRow)
case class CheckpointEvent(position: BinlogPosition)
```

(see also [Use `sealed trait` for union types](#use-sealed-trait-for-union-types))

This way, the types tell you which event you're dealing with. Anything acting on a `BinlogEvent` _must_ explicitly pattern match to deal with each possibility, but you can also write functions accepting only a `DataEvent` or `CheckpointEvent` for code paths where those have been separated, and the compiler will guarantee you're doing that correctly.

**Note:** We could have written the above data type as a simple `Either[BinlogRow,BinlogPosition]`. For data types used in one or two functions that's probably fine, but if the type is frequently used (or if either side is more than just a single field), the above `sealed trait` version is simpler to refer to and more extensible.

## Use `sealed trait` for union types

Union (disjoint union) types are types that can be exactly one of several types. 

There are two types of union types:
* Union types that contain values.
* Union types that don't contain values. These are often called *enumerations* or simply *enums*.

For types that contain values, the main reasons for representing with sealed traits are:
* To correctly represent the data by construction, providing static state validation. 
    As long as each of the subtypes are correctly represented, so too will the sealed trait as a whole.
    See [Try to make data correct by construction](#try-to-make-data-correct-by-construction) for an example of this. 
* Sealed traits are exhaustively checked when pattern matching. Missing a case will generate a error at compile time.

The drawback of using sealed traits for union types is that each type must be a subtype of the sealed trait.
This doesn't work out-the-box for pre-existing types. The workaround is to create a wrapper for each of the pre-existing types.

For example, suppose you have existing types `A`, `B` and `C` and you wish to create a union type `D`. 
This can be represented as:
```scala
sealed trait D
final case class WrapA(a: A) extends D
final case class WrapB(b: B) extends D
final case class WrapC(c: C) extends D
```

There are two special cases of this pattern:

* **`Either`**

    Suppose there are two distinct states, in which one of them refers to the "happy path" or "expected path" and the other refers to a "unhappy path", in which an error or otherwise unexpected behaviour occurs.
    In this case it is common convention to represent this with `Either[E, A]`, where `E` is the error type.
    
    The wrapper classes are `Right(value: A)`, a container or wrapper for the successful state, and `Left(e: E)`, a wrapper for the error state.
    
    `Either` is a so-called *right biased* data structure, with `map`, `flatMap` and other combinators that operate on the right hand side. These are useful for composing successful operations.
    
    `Either` should preferably be used to represent right-biased data.
    
*  **`Option`**

    A related special case to this is `Option[A]` which can be considered a union of a value of type `A` and `null`.
    
    The wrapper class for the non-empty state is `Some[A]`. The empty/null state is represented by the singleton `None`.

### Enumerations

Enumerations deserve consideration on their own. 

There are 3 alternatives:
* Derive your enumeration type from `scala.Enumeration` 
* Use Java `enum`s.
* Use sealed traits with case objects.

Of these options, sealed traits provides the best pattern matching support. This is because each 
value is a separate type which can be matched against, and is verified exhaustively in pattern matching expressions.

For example
```scala
sealed trait Colours

case object Red extends Colours
case object Green extends Colours
case object Blue extends Colours

def getName(c: Colours) = c match {
  case Red => "red"
  case Green => "green"
}
```

generates the following at compile time:
```text
Warning:(6, 29) match may not be exhaustive.
It would fail on the following input: Blue
  def getName(c: Colours) = c match {
```

Of the other alternatives, we don't recommend using Java enums unless specific interop with Java is required. 

When using `scala.Enumeration` the specific type is erased, which means it can't be pattern matched against.

Because of better type safety and pattern matching support, we prefer to use sealed traits to represent enums.

## Aim for pure functions
A pure function is side-effect free, plus the result does not depend on anything other than its inputs.  Our functions
should also be total. For any inputs the type system allows they should not throw an Exception (returning a value
representing an error is fine).

For a detailed overview of why pure functions are beneficial, see https://dzone.com/articles/scala-best-practices-pure-functions-1.

Exceptions:
- Note we often treat logging as an impure side effect which we do not care about testing.

## Favour lazy effects over eager effects
We should favour lazy effects over eager effects. `scala.concurrent.Future` is eager.  This results in limitations like:
- It is not referentially transparent. The following code blocks behave differently when ideally they should not.
```scala
val f1 = Future.failed(new RuntimeException)
val f2 = Future { saveToTheDB() }
for {
  _ <- f1
  _ <- f2
} yield ()
```
```scala
for {
  _ <- Future.failed(new RuntimeException)
  _ <- Future { saveToTheDB() }
} yield ()
```
- Futures are less efficient as every `map` / `flatMap` must be requested on the execution context and may be run on a
different thread.
- Because Futures are eagerly evaluated, it's impossible to manipulate the value post construction.  Eg the following
will never be possible with Future:
```scala
saveToDB().delayExecution(1.second)
```
- Because of the eager evaluation, the `ExecutionContext` needs to be implicitly wired everywhere.

Alternatives which address the limitations of Future mentioned above.
- [Monix Task](https://monix.io/) - Currently the default solution of choice for Goanna projects
- [Cats effect IO](https://typelevel.org/cats-effect/datatypes/io.html)
- [ZIO](https://zio.dev/)

The alternatives above also offer:
- Resource management
- Task cancellation

Exceptions:
- We generally haven't worried about treating logging as an effect.  In Goanna code bases, logging generally occurs as a
side effect not represented by the type system (but if you wish to model as an effect, that effect should ideally be
lazy).
- You might favour `Future` if the libraries used by the codebase are tightly coupled to `Future` (eg Gourmand, Akka
Streams).

## Resource acquisition

Resources should always be bracketed in some way. We should use cats-effect [Resource](https://typelevel.org/cats-effect/datatypes/resource.html) type to perform automatic cleanup regardless of effect type (sync, Task, etc)

For resources that are only used synchronously (and we don't want to depend on `cats-effect`), a `use[T](block: Resource => T)` method implemented as a simple `try { ... } finally { ... }` is fine.

When working with `Future`s, we can't use `Resource`. So in that case we can either:
- Migrate the usage to `Try` or equivalent (preferred)
- Implement a custom `use[T](block: Resource => Future[T])` function which does the cleanup in an `onComplete` block.

See also [Process termination](#process-termination).

## Mutable state

Mutable state includes both:

- `val`s referring to a mutable data structure (e.g. `val items = ListBuffer[String]()`)
- mutable variables (`var`), referring to immutable (or mutable) data structures

Firstly, when adding either of these, try to get someone to talk you out of it.

If you do use mutable state:

 - It's on you to prove that it's thread safe, or that it will never be accessed by multiple threads
 - Never expose your mutable state outside of its owning class

Exceptions:

 - Mutable state within a function (where it's either thrown away or converted into an immutable version by the time the function returns) is acceptable, and can occasionally be more readable than functional approaches.
 - Trusted libraries where mutability is a feature (e.g. scaffeine)

## Convert from unsafe Java models to Scala case class
Java models may not convey through types which fields are optional.  For example, a Kafka ConsumerRecord has a `key` and
`value` which can potentially be `null`.  Using this Java model throughout the codebase introduces many places that we
could accidentally introduce a `NullPointerException`.

In this example, the service should define a model which reflects its domain.  If it expected the `key` and `value` to
always be present, it would define something like:
```scala
case class SafeKafkaRecord(key: String, value: String)

def transformRecord(consumerRecord: ConsumerRecord): Try[SafeKafkaRecord] = ???
```
`transformRecord` would return Failure if the key or record was absent, but this concern would be isolated to a single
place in the codebase.

Alternatively, if the service expected the key and value to be optional, it would define.
```scala
case class SafeKafkaRecord(key: Option[String], value: Option[String])

def transformRecord(consumerRecord: ConsumerRecord): SafeKafkaRecord = ???
```
Note now that `transformRecord` never fails, but all other service code is enforced to handle that the `key` and `value`
are optional.

|Pros|Cons|
-|-
|Prevents errors|Runtime overhead (slight)|
|More explicit data types| |
|Better readability and easier to reason about the code for maintenance| |

## Process termination

Shutting down is hard. Here are some things to ensure:

 - Every thread must have an `UncaughtExceptionHandler` which causes process termination ([sample in kafka-bus](https://github.com/zendesk/kafka-bus/blob/fc737683fe3f8701bfc80a99ac0b969ecf039419/kafka-router/src/main/scala/com/zendesk/eventbus/router/CustomScheduler.scala#L22-L42)). Why? The default JVM behaviour is to log and continue, which can cause tasks / futures to never complete, amongst other undefined / broken behaviour.

 - To ensure orderly shutdown, prefer using monix's `TaskApp`. This cancels the app's task gracefully on JVM termination. This is achieved via the underlying [IOApp](https://typelevel.org/cats-effect/datatypes/ioapp.html#cancelation-and-safe-resource-release), so that may be useful if we're unable to use TaskApp for some reason.

 - Some libraries cannot be relied upon to shut down in an orderly fashion, they may get stuck forever in rare cases. Where this is the case, we can use a failsafe shutdown mechanism like [this one in Maxwell](https://github.com/zendesk/maxwell/blob/1f7fecb24bcc24c1587541d8831c165efe52c01a/src/main/java/com/zendesk/maxwell/MaxwellContext.java#L175-L227). This could probably be simplified for other apps.

# Design goals
> Aim for pure functions, favour lazy effects, use the type system to prevent errors, avoid mocks.
use code patterns e.g. make first-class test implementations, case classes, sealed traits

## Make first-class test implementations

This is a useful strategy for [avoiding mocks](#avoid-mocks): creating real, simple classes which implement interfaces in a particular way.

- [FakeClock](https://github.com/zendesk/zendesk_core_scala/blob/8f90cfd77a2bbd36c77c067cc6d41038faa7cde7/zendesk-testkit/src/test/scala/com/zendesk/testkit/FakeClock.scala):
  A `Clock` implementation where the "current time" is controlled by explicitly modifying it
- [TestStatsDClient](https://github.com/zendesk/zendesk_core_scala/blob/8f90cfd77a2bbd36c77c067cc6d41038faa7cde7/zendesk-metrics/src/main/scala/com/zendesk/metrics/testsupport/TestStatsDClient.scala):
  A `StatsD` implementation which accumulates statistics and exposes them for use in assertions
- [ManualConsumer](https://github.com/zendesk/maxwell_smarts/blob/a38a749f7f36f8046a3d1a364c5c1314d64b6218/src/test/scala/com/zendesk/maxwellsmarts/support/KafkaConsumer.scala#L41-L60):
  A kafka `Consumer` instance where the records which will be returned by a `poll()` are mutated by explicitly `push()`ing them.

Making explicit implementations which are used for tests has many advantages over mocking.

### Benefits:

#### Better reuse

They encourage code reuse, since they aren't tied to a particular test (unlike using a mock)

#### Less fragile tests

They also decrease coupling. Test implementations are built using required information -- you pass them what information they _need_ to do their job. Whereas mocks frequently involve a mix of required information and incidental information.

For example, in the case of `TestStatsDClient`, you can assert on the `Count` values which were recorded. With a mock, your test would have to specify _which_ of the dozen `count`-related overloads is actually being invoked. Similarly for mocks which implement behaviour, it's easy to implement a mock which is fragile and inconsistent, by (say) returning different results from `poll()` and the `poll(d: Duration)` overload.

#### Improve the quality and readability of test assertions

Again using the `TestStatsDClient` example, you can assert on the `Count` values which were recorded, and because it's just data, you're free to `map() or `filter()` over the results in order to assert on a subset of information (e.g. discarding the tags, or sum up the total counter values across multiple calls). This requires no mock-specific logic to implement or read, it's just manipulating collections.

With a mock, similar tests would need to use advanced matching APIs (which many people aren't familiar with, since they're so niche) in order to (say) match tags regardless of ordering, or match a subset of calls, etc.

## Pull out pure functions into companion object

This is a workaround in order to test individual parts of a class which has dependencies unrelated to the functionality you're testing.

**Note:** Before applying this pattern, try to break up the object in question into more focussed classes which _can_ be easily tested.

Say `class Foo` takes 5 constructor arguments (`a,b,c,d,e`), but some functionality (`doThing()`) only needs to access `a` + `c`.

If you want to test `doThing()` but it's difficult to construct a `Foo` in tests, you can pull out a static function as a form of lightweight dependency injection:

```
object Foo {
  def doThing(a, c)(args)
}
```

The `foo` instance can get a function via `val doThing = Foo.doThing(a,c)`. The main benefit is that tests can invoke this specific functionality in isolation without needing mock instances for `b`, `d`, `e`.

Here's a concrete example in [Maxwell Smarts](https://github.com/zendesk/maxwell_smarts/blob/a38a749f7f36f8046a3d1a364c5c1314d64b6218/src/main/scala/com/zendesk/maxwellsmarts/kafka/Processor.scala#L167-L174)

## Create application config and pass it in where required
Instead of reaching for a global config, we should aim to create a config object and pass values where required.

|Pros|Cons|
-|-
| More explicit | Function arguments may get large* |
| Polymorphic | |
| Pure Functions | |
| Simple to write tests | |

**Global Config example**
```scala
// Global config defined, simple
object GlobalConfig {
  val producer = sys.env.get("PRODUCER")
  ...
}

// Reaches into global config, implicit dependancy on GlobalConfig
def someFunction(): Unit {
  // If we wanted to user another configuration, we'd have to change the GlobalConfig
  val producer = GlobalConfig.producer 
  useProducer(producer)
}
```

**Passing in example (explicit)**

```scala
// Config class
case class Config(producer: Producer, ...)

// Create Config class
val config = Config(sys.env.get("PRODUCER"), ...)

// Call, passing what is only required
someFunction(config.producer)

// Explicit dependancy on producer
def someFunction(producer: Producer): Unit {
  useProducer(producer)
}
```

**Polymorphic & Simple to write tests**

This makes each class less coupled to a specific configuration. It becomes much easier to pass in custom configurations for tests or polymorphic versions of the application.

For example, imagine an application running in two modes. It will need two different configurations. By passing in a configuration, we only need to create two configuration objects. If we used a global configuration, we'd need logic inside the calling class to decide which configuration to choose from, or a global configuration that could give one of the possible configurations, which can get bloated.

By having the functions pure, we can write tests without configuring the whole environment.

**Pure Functions**

The output depends solely on the inputs. With global configs, this isn't the case. See [Aim for pure functions](#aim-for-pure-functions) for more details.

**Function arguments may get large**

Depending on how nested function calls are (large call stack), functions may end up having lots of dependencies in the arguments. This should be avoided anyway due to coupling concerns.

Code should preferably be easy to understand, rather than easy to do. Although there's more arguments (harder to do), it makes the function more precise (easier to understand).

## Prefer configuration settings to live in the codebase
Some configuration needs to change between environments. For example, hostnames, database passwords, connection strings.

Other configuration will be consistent across environments. For example, we are unlikely to require a different database connection timeout between two production pods.

If configuration doesn't need to change between environments, prefer including that configuration in the codebase as opposed to Samson environment variables.

Don't proactively include the ability to override a configuration setting with an environment variable. If you do, anyone looking at that variable will now need to look in multiple places to reason about the setting.

Experimentation / test changes can be made in Samson, but once the config has stabilised it should be moved back into the project.

|Pros|Cons|
-|-
| Look in a single place to understand config | Can't arbitrarily change config in just one environment |
| Config changes go through the PR process. So they are reviewed and have a commit comment describing why the change was made. | Config changes must go through build pipeline, might be inconvenient if other changes on master have yet been deployed. |

## Single scheduler between Futures and Tasks

Generally, we'd like to use one Scheduler. We should be consistent with which scheduler to use and should be passed in as an argument. It makes the function easier to understand and makes it more testable.

```scala
// Bad, we shouldn't use these global implicits outside of tests.
// When we do use them, be consistent and choose one.
import monix.execution.Scheduler.Implicits.global
import scala.concurrent.ExecutionContext.Implicits.global
```

Instead, we should aim to share single scheduler, passing them in as arguments.

Generally this can be done with extending TaskApp:

```scala
// Good
object Main extends TaskApp {
  override def run(args: List[String]): Unit = {
    Service.doAsyncStuff(scheduler) // `scheduler` is defined from TaskApp
    ...
  }
}

object Service {
  // Require the scheduler, instead of importing the global implicits
  def doAsyncStuff(implicit scheduler: Scheduler): Future[Unit] {
    Future {
      println("hello")
    }
  }
}
```

There are cases where we might need multiple schedulers:

- Cheeky libs or blocking IO
- To provide isolation between different domains (e.g. to deal with noisy neighbor problems or badly behaved libraries)
- If there's blocking IO, it should be on a separate scheduler

## Project structure consistency across our repos

A consistent project structure across repos makes it easier for newcomers understand the codebase, know where to look for things, and get productive faster. 
It enables more fluid context switching when working on multiple codebases.

If you use the [Scala starter [Giter8 template project](https://github.com/zendesk/zendesk-scala-api-template.g8) to create a project, it bootstraps a project according to Zendesk standards.

This is the recommended project structure for single module projects. Try to adhere to this as much as possible. 

```
base                build.sbt, Dockerfile, README.md etc.
  |-- kubernetes    Kubernetes manifests
  |-- project       build.properties, plugins.sbt and other sbt files
  |-- scripts       utility scripts
  |-- src
       |-- main     application source code
       |-- test     application test code
```
This structure is expected to evolve somewhat over time to accommodate changes such as different build and deploy mechanisms (e.g. using Dhall for CI).
  
The following are not expected to change:
* The project consists of `src` (source) `main` and `test` folders.
* All package names start with `com.zendesk.<projectname>`.
* The project directory structure matches the package names.
* The `test` package structure should mimic the `main`, enabling unit tests on package private classes.

In addition, the following guidelines apply:
* Packages should not get too large, and should be proactively refactored to avoid this.
* Prefer a relatively flat package structure and avoid too deep nesting.
* Keep the base package light.
* Make declarations package private as much as possible. 
 In Scala this is done for example:
   ```scala
   package com.zendesk.myapp
   private [myapp] class MyClass { ... }
   ```
   restricts access to `MyClass` to the package `com.zendesk.myapp`.
                                                                              
* If working in a multi-module project, the package name should start with `com.zendesk.<projectname>.<modulename>`. 
This ensures that package names are not reused between modules *(see [package objects](#package-objects) for problems with this)*.

### Package-by-feature

In cases where it makes sense to do so, arranging packages according to package-by-feature is suggested over package-by-layer. 

This specifically applies to cases where the notion of features is prevalent, such as a web service. 

In this case there is at a bare minimum a controller layer, a domain service layer, and a data access layer. 
Adding a single feature may correspond to a CRUD endpoint, or a set of endpoints for resource, and their associated implementation. 

Packaging-by-layer involves changes to packages for each of these layers.

Packaging-by-feature involves changes to a single package. This speeds up development, and makes changes easy to follow (for example in pull requests).

### Package Objects

Be careful with package objects as they are closed: 
Declarations in package objects look like top level objects, but unlike top level objects, you cannot extend the package object outside the single file it's defined in.

Each package can contain only one package object. As this restriction is only enforced for a single module, it is possible to inadvertently end up with two package objects with the same package name. 
One of these will be overwritten in the classpath in a non-deterministic way. 
This may only be discovered at runtime.

For this and other usability reasons, use package objects judiciously.

They are deprecated from Scala 3.
 
## Different types of testing

### Unit tests
These tests should be performed as part of CI.

Unit tests test the smallest amount of publicly exposed code. Private implementation details do not
need to be tested but can be done for good measure.

Testing private code can be done by exposing it at the package level.
```
private[packageName] def complexFunction: Int = ???
```

Unit tests are not defined or limited by technology. If the test requires, you may need a datastore
to test its correctness.

For example: you may need a MySQL database to test SQL queries;
you may need a Kafka instance to test publishing or consumption of messages.

In either of those cases, the preferred method, if possible, is to use a test container which runs the service. The service in the test container should mirror the version and configuration in production.

### Integration tests
These tests should also be performed as part of CI.

Integration tests test multiple publicly exposed code together, typically with complex interaction.

Similar to unit tests, this is defined by surface area of publicly exposed code, not technology. Use the technology necessary to perform this test.

### End to end tests
Theses tests are typically performed on our staging and sandbox environments.

These are usually performed once as verification.

As a rule of thumb, think twice before creating a automated end to end test. Err on the side of an
integration test and rely on metrics and monitors to determine the health of your code.

For example, the Global Event Bus and Escape both use heartbeats, live in production, to make sure
the application is working correctly. Errors should be caught in staging.


## Avoid mocks
**Mock** is a controversial topic/tools involved in almost all languages nowadays; This guide would focus on the ways we could do to avoid mocks instead of going deep to why we should avoid mocks, for more details, this is a good article to read: https://www.rea-group.com/blog/to-kill-a-mockingtest
 
 
### Define a Lean and precise function contract to avoid mocks
If a test requires a data object used by some functions to be mocked, it is most likely a smell for an inappropriate definition of a function contract or fat dependencies; 
 
e.g.
 
```scala
case class KafkaConfig(
  ...
  many items inside 
  ...
)
 
object HealthCheck {
  def apply(config: KafkaConfig): Future[Either[IOERROR, DONE]] = {
    kafkahost = config.kafkahost
    port = config.port
 
    ....
    //do something
    ....
  }
}
 
object TestCase {
  val config = mock[KafkaConfig]
  when(config.kafkahost).thenReturn("localhost")
  when(config.port).thenReturn("9092")
 
  HealthCheck.apply(config)
 
  .....
}
```
:point_up: This is a typical example about **mocking a data object** just because it could be too tedious to be initialised in the test; it indicates the function `apply(config: KafkaConfig)` should be able to further optimise to reduce the scope of dependencies it needs from kafka config; and obviously, it only requires **kafkahost** and **kafkaport** in this case; 
 
A refactored version without mocking involved in the test could be like:
```scala
object HealthCheck {
  def apply(kafkaHost: String, kafkaPort: String): Future[Either[IOERROR, DONE]] = {
    ....
    //do something
    ....
  }
}
 
object TestCase {
  val kafkaHost = "localhost"
  val kafkaPort = "9092"
 
  HealthCheck.apply(kafkaHost, kafkaPort)
  .....
}
```
Now the health check function would specify the dependencies it needs without requiring the entire fat kafka config; and from the test point of view, mock is avoided;
 
### Define loosely coupled modules/units to avoid mocks
Deeply coupled code could be shown in many different forms, a typical smell for this type of problems could be detected when you have to mock a service inside another service so that you could continue on your test; E.g.
 
```scala
class Processor(private val kafkaConsumer: KafkaConsumer[String, String]) {
  //do something
}
 
object ProcessorTest {
  val kafkaConsumer = mock[KafkaConsumer]
  when(authService.poll()).thenReturn("a messgae")
 
  val processor = new Processor(kafkaConsumer)
  // do assertion
} 
```
:point_up: as a **KafkaConsumer** is a concrete class containing all the detailed implementations(https://kafka.apache.org/20/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html), in order to test the **Processor**, the only way is to mock **KafkaConsumer**; how about we replace the concrete class with an interface, e.g. **Consumer<K, V>** (https://kafka.apache.org/20/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html)
 
```scala
class Processor(private val kafkaConsumer: Consumer[String, String]) {
  //do something
}
 
object TestConsumer extends Consumer[String, String] {
  //implement all the contracts you need
}
 
object ProcessorTest {
  val processor = new Processor(TestConsumer)
  // do assertion
} 
 
```
:point_up: now as the processor decouple itself from **KafkaConsumer** by just requiring a set of contracts from **Consumer[String, String]**, we could provide a custom simple implementations of these contracts to processor for testing purpose without mocking anything;
 
### Test IO with real resources instead of mocking
Another common cause for mocking is to mock IO, e.g. db/file operations; mocking this kind of behavior would not provide any guarantees on your code; instead of mocking, real resources should take place to provide real tests. And if things become too hard, which could be a strong indicator that the actions declaration is deeply coupled with the actions operations; otherwise, for IO test, a docker container running db or a tmp dir/file should be enough to test all the IO behaviors;

### See also: [Make first-class test implementations](#make-first-class-test-implementations)

## Limit the usage of implicit parameters

There are several common usages of implicits in Scala:

1. Implicit methods
2. Implicit parameters for dependency injection
3. Implicit classes for extension methods
4. Typeclasses

In [EPFL's Dotty / Scala 3 overview](https://dotty.epfl.ch/docs/reference/overview.html), EPFL mentions one of their goals is to "Tame powerful constructs such as implicits to provide a gentler learning curve". 

The creator of Scala, Martin Odersky himself, [has admitted that implicits have not been successful](https://contributors.scala-lang.org/t/principles-for-implicits-in-scala-3/3072):

> If implicits are so good why are they not the run-away success they should be? Why do the great majority of people who are exposed to implicits hate them, yet the same people would love Haskell’s type classes, or Rust’s traits, or Swift’s protocols?

Later, he explains:

> In fact it’s better not to think of them as parameters at all, but rather see them as constraints.

In our Scala codebases, we should aim to use implicits as "constraints", i.e. for typeclasses.

### Implicit methods

The usage of implicit methods can lead to surprising behaviour. 

```
  implicit def intToList(num: Int): List[Int] = List(num)

  def add(x: Int, y: Int): List[Int] = x ++ y
```

In this example, `Int`s are implicitly turned into `List`s. You can do `List` operations` on `Int` as a result. This is especially bad if the `implicit def` is defined elsewhere and imported into scope accidentally. 

The preferred alternative is to explicitly call methods that you need. 

### Implicit parameters for dependency injection

Implicit parameters can be used for injecting dependencies. In the following example, `Metrics` is implicitly injected.

```
class Controller(service: Service) {
  def getUser(userId: UserId)(implicit metrics: Metrics) = { 
    // do something with `service`
    // do something with `metrics`
  }
}
```

And then at call site:

```
implicit val metrics = new Metrics()
val service = new Service()
val controller = new Controller(service)
controller.getUser(UserId(123)) // Note there is no need to provide `Metrics`
```

The implicit nature of this approach can lead to confusion when reading the code, particularly at call site. Therefore, we recommend limiting the usage of such implicit parameters to the ones that are already commonly used in existing Lizards codebases:

1. `ExecutionContext`
2. `Scheduler`, `Timer`, `Clock` (from [Monix](https://github.com/monix/monix))
3. `Metrics`
4. `StatsDClient`

Certain libraries are using this technique (see: [Monix](https://github.com/monix/monix)). We should be cautious not to introduce too many more of these examples.

### Implicit classes for extension methods

@szoio to expand on

### Typeclasses

Typeclasses is a design pattern that can be achieved using implicits in Scala. It is not viable to program using typeclasses without using implicits, because of the recursive nature of some typeclasses. Therefore, using implicits for achieving this design pattern is encouraged.

```
import io.circe.syntax._

object Person {
  implicit val encoder: Encoder[Person] = Encoder.forProduct1("name")(p => p.name)

  def convertPersonToJson(person: Person): Json =
    person.asJson // asJson takes an implicit `Encoder[Person]`
}
```

In the example above, `person.asJson` accepts an implicit `Encoder[Person]`, which relies on an implicit `Encoder[String]` to encode the `name` to a `String`. 

Another example:

```
def putStrLn(x: Int): Task[Unit] = Task(println(x))

List(0, 1, 2).traverse(x => putStrLn(x)) // Task[List[Unit]]
```

A lot of combinators in Cats, such as `traverse`, `mapN`, `sequence`, `combineAll`, etc. rely heavily on implicit typeclass instances. In the example above, the compiler will search for an implicit `Applicative[Task]` and an implicit `Traverse[List]`. It is not viable to replace these implicit parameters with explicit parameteres.

Here is a list of the more common typeclasses that you might see day-to-day in Scala codebases that should be used as implicit parameters:

1. `Decoder` and `Encoder` (from [Circe](https://circe.github.io/circe/))
2. `Async`, `Sync` (from [Cats Effect](https://typelevel.org/cats-effect/typeclasses/))
3. `Functor`, `Applicative`, `Monad`, `Semigroup`, `Monoid` (from [Cats](https://typelevel.org/cats/typeclasses.html))

## Techniques for reducing unnecessary diffs

Large-scale changes are often easy and safe in Scala, however they're still disruptive, particularly when it comes to code review and merge conflicts. Reducing the frequency of unnecessary diffs is useful, and can often clarify intent.

### Wrapper functions for the class / method under test

Tests are repetitive by nature. Typically it's worth making at least one wrapper function per test class which either builds or runs the thing that most of the tests exercise. Unlike production code, this should use default arguments wherever possible so that tests only need to specify what they care about (using named arguments).

[Here's an example of applying this technique](https://github.com/zendesk/mysql-mover/pull/233/files#diff-51e2789d706d47c40d0f9b36c4d1a73e).

# Zendesk
> Zendesk specific standards

## Don't use `zendesk/sbt-base-settings`
`zendesk/sbt-base-settings` is a SBT auto-plugin that provides common settings for our Scala projects. These settings minimize boilerplate code in build.sbt that can obscure the important bits. Most importantly, they enable additional compiler warnings that help eliminate bugs.

However it contains a lot of features and has some strong opinions. For alternatives that allow an opt-in approach to relevant features, refer to project [guidelines](https://github.com/zendesk/zendesk_sbt_base_settings#modular-alternatives).

## Continuously deploy to staging
Our projects should be set up to automatically deploy code changes to the staging environment. A large number of product teams rely on the services we provide, so it is important to deploy changes quickly so that any issues or problems can be caught as soon as possible by our Datadog monitors and alerts, and by other teams' automated testing.

### Using Samson
Stages in Samson can be configured to **Automatically deploy new releases**. For stages deploying to Pod 998 and 999 this setting should be ticked.

A release tag is automatically made by Samson if **Release branch** and **Release source** are configured in the project settings page. The recommendation is to set **Release branch** to `master` and **Release source** to your preffered triggering event (code commit, pull request, successful CI run, etc).

### Using Spinnaker 
A Delivery Pipeline can be configured to deploy Kubernetes manifests to our Pods based on Automated Triggers. The typical setup is to use a `Git` type trigger that runs when commits are pushed to the `master-deploy` branch. 

Github Actions can be set up that do the following:
- build Docker image for the change
- push image to Artifactory
- generate Kubernetes manifests on the fly
- bump the version/release in git
- publish the change to git

The result is a commit to the `master-deploy` branch with a corresponding Docker image in Artifactory ready for deploying to Kubernetes.

# Scala Libraries
> Justifications and comments on libraries we use

## Justification for `Cats`
There are many FP approaches we are taking in our Scala code bases.
- [respresenting failures as values](#use-tryt-instead-of-exception)
- [immutable data structures](#use-immutable-data-structures)
- [modelling effects](#favour-lazy-effects-over-eager-effects)

Cats is a:
> Lightweight, modular, and extensible library for functional programming

The original library in this space was Scalaz, but Cats quickly became the defacto standard due to more approachable
documentation and a healthier open source community.

Within the Zendesk community, Cats was clearly more favoured than Scalaz.
(See [2019 Zendesk Scala developer preferences survey](https://zendesk.atlassian.net/wiki/spaces/LS/pages/737577490/Zendesk+2019+Scala+Developer+preferences))

## Justification for `Monix`
See [Favour lazy effects over eager effects](#favour-lazy-effects-over-eager-effects) for why we need Monix in the first
place.

We decided on `Monix` over `ZIO` due to the maturity of the library in March 2019.  `Monix` also provides a reactive
streams implementation and an elegant API for time based operations (eg `Task(someEffect()).delayExecution(1.second))`).
Its `TestScheduler` makes testing of time base operations deterministic and fast.

## Don't use the twitter stack
In the [2019 Zendesk Scala developer preferences survey](https://zendesk.atlassian.net/wiki/spaces/LS/pages/737577490/Zendesk+2019+Scala+Developer+preferences),
Twitter Future was universally unpopular (being only more popular than blocking IO).  It's painful because the classes
share the same names as all the Scala implementations, making it very confusing.

At the time Twitter Future was released, it did improve on the Scala Future, but it's still based on an eager effect
model.  See [Favour lazy effects over eager effects](#favour-lazy-effects-over-eager-effects) for why this isn't ideal
and for alternatives.

## Use `===` rather than `==` for equality

In Scala it’s possible to compare any two values using == (which desugars to Java equals). This is because equals type signature uses Any (Java’s Object) to compare two values. This means that we can compare two completely unrelated types without getting a compiler error. The Scala compiler may warn us in some cases, but not all, which can lead to some weird bugs.

Instead, the cats `Eq` [typeclass](https://typelevel.org/cats/typeclasses/eq.html) is used to enforce typesafe equality checking.

`if ("hello" == 1)` will never be true, so let the compiler tell us we have done something wrong.

### Operator `===`

- Needs to be imported e.g. `import cats.implicits._`
- Supports primitive types e.g. String, Boolean and Integers etc.
- Non primitive data types will need to implement `Eq` 
  - Case classes can use use the `Eq.fromUniversalEquals` helper function, which defers to the `.equals` (implemented by default for case classes).
  - Other data types will need to implement `Eq` explicitly.

Implementing `Eq` for your case classes

```scala
package com.example

import cats.Eq

case class MyType(id: String, name: String)

object MyType {
  implicit val eqMyType: Eq[MyType] = Eq.fromUniversalEquals
}
```

Usage

```
MyType("x-1", "blue") === MyType("x-1", "blue")
// result: true
```

### Test framework

We use [scalatest](http://scalatest.org/), with the `FunSpec` style which is terse and familiar to those familiar with `rspec`. Scalatest has some issues, in particular:

 - it's a very big interface (so many matchers, different testing styles, etc)
 - doesn't integrate with `cats`' typesafe equality

We're open to something better, but we should stick to this until we decide on a better framework.

# Hygiene
> Miscellaneous stuff you should know and are generally best practices.

## No casting `asInstanceOf`

Type casting is a conversion of a value from one type to another. The compiler will not be able to guarantee type safety in this case and this is often completely unnecessary. Here is an example:

```
sealed trait User
case class SignedInUser(email: Email) extends User
case class AnonUser(cookie: Cookie) extends User

def handleUser(user: User): Email = {
  user.asInstanceOf[SignedInUser].email
}
```

This piece of code will fail at runtime if the input is an `AnonUser`. Prefer using pattern matching instead:

```
def safeHandleUser(user: User): Option[Email] = {
  user match {
    case SignerInUser(email) => Some(email)
    case AnonUser(_)         => None
  }
}
```

## Use `if` for booleans, not a pattern match
While pattern matching is amazing, it doesn't improve code clarity for boolean expressions.

Avoid:
```
myValue.someBooleanProp match {
  case true => ...
  case false => ...
}
```
Use:
```
if (myValue.someBooleanProp) ...
else ...
```

## Keep dependencies up to date

It is important to ensure that the libraries and plugins used in our projects are kept up to date. This ensures we are up to date with security vulnerabilities and bug fixes. The libraries are often specified in `build.sbt` and the plugins are often specified in `project/plugins.sbt`.

In a similar vein, ensure that the `sbt` version in `project/build.properties` and the Scala version in `build.sbt` are kept up to date too.

## Use dots and brackets `foo.bar(1)` instead of `foo bar 1`.
- DSLs OK (e.g. specs etc)
- Have noticed that scalatest doesn’t follow above convention
## Use `Try[T]` instead of exception
- “Fatal” errors can be exceptions
- Either[SomeErrorType,T] is OK too

## Never use `Await.result` outside the main thread (or tests)

Consider keeping the `Future` as long as you can and only run it _at the end of the world_. 
This allows you to compose multiple `Future`s (e.g. using `.flatMap`) easily in the majority of your program. 

This principle applies to `monix.eval.Task` and other effectful data types like `cats.effect.IO`. 
Instead of calling `runSyncUnsafe` (or its variants), you should chain and compose your `Task` or `IO`
and then run it once _at the end of the world_ if necessary. The preferred approach is to give the
`Task` or `IO` as a value to the framework to run (`monix.eval.TaskApp` and `cats.effect.IOApp` respectively),
so you don't have to do it.

By using `Task` or `IO`, you also maximise [referential transparency](https://en.wikipedia.org/wiki/Referential_transparency). [Read more about lazy effects](#favour-lazy-effects-over-eager-effects).
in your code and you can safely refactor your code without accidentally introducing a bug due 
to lack of referential transparency.


## Thread creation (including as part of thread pools):
- always set an UncaughtExceptionHandler which terminates the process
- always `setDaemon(true)`
- Always use a named thread factory (assigns meaningful name)
## Favour composition over inheritance
- https://github.com/zendesk/zendesk_health_check_scala/pull/62
- Interfaces are fine

## No default parameter values in src/main which are only for tests
- Make a local `def` or fixture in tests instead

## Exhastively pattern match on `Option`
- No `case _ =>` when pattern matching on Option 
- or similar where the compiler could guarantee exhaustiveness

## No `Seq` - Use `List`, `Chain` or `Vector`
From https://github.com/typelevel/cats/issues/1222#issuecomment-234605211
> The short answer is that Seq doesn't tell you much. Unlike List or Vector, you don't know what the performance
> characteristics of a Seq instance are going to be, so you don't necessarily know the best way to implement methods in
> the type class instance. It may be mutable, in which case mySeq.map(identity) can't simply be rewritten as mySeq
> without potentially changing the meaning of your program.

If working with `Seq`, you can't evenly safely pattern match with `case head :: tail =>` as that would couple to the
implementation being `List`, but this would not work with other implementations of `Seq` like `Vector`.

## No null
Assigning to `null` should be avoided at all costs.
Any value that could be `null` has to be explicitly checked before being dereferenced. The compiler is unable to enforce that this check happens when required.
Skipping a check will lead to you reaching for the data (or: dereference) that resides… exactly, nowhere, and `NullPointerException`s will be thrown

### Initialise with Option instead of `null`
A main replacement for null values is to use the Option/Some/None classes.
For example one of the places where a null value can silently creep into your code is when a field in a data type may not exist at one point.
```scala
case class User(name: String, maybeJob: Option[Job])

val personWithNoJob = User("Bob", None)
val personWithJob = personWithNoJob.copy(job = Some(Engineer))
```
### Never return `null` from methods
Instead of null return an Option. Also we should check the return value of a function or method that might return null and can convert to an option with Option.apply.

```scala
def divide(dividend: Int, divisor: Int): Option[Int] =
  if (divisor == 0) None <--- Instead of Null  
  else        Some(dividend / divisor)
```

## Use immutable data structures

Immutable data structure are never updated in-place, but instead always yield a new updated structure.

It removed side effects making the code deterministic and easier to reason about locally. It allows for effortlessly parallellism that is thread safe.

Although Scala splits its collections into `immutable` and `mutable` packages, its Predef object automatically puts the immutable data structures (such as Map and Set) into the default environment. Meaning creating a new Map or Set, automatically uses the immutable structures.

## Prefer `val` over `var` declarations

`val` defines an immutable constant which cannot be modified once declared and assigned.

`var` defines a mutable variable, which can be modified or reassigned.

using `val` to declare immutable constants will remove the chance for subtle bugs to creep into the system. Refer to [Pure Functions](#aim-for-pure-functions)

> Prefer vals, immutable objects, and methods without side effects. Reach for them first. Only use vars, mutable objects, and methods with side effects when you have a specific need and justification for them.

## Don't let serialization corrupt your core model just to match the shape of expected json input / output
- E.g. foo_bar instead of fooBar to match JSON requirements
- You can, but only at the end, and explicitly as a JSON target for the domain version of that object

## Don't put multiple classes in one file
- unless it's a companion object or an ADT

## Delete merged / unrequired branches
- better yet, setup github to do this automatically

## Wrap a Case Class if you want a free toString / equals / hashcode

Sometimes you want an object which implements equality / toString, but you don't want to implement that yourself because that's [not trivial](https://stackoverflow.com/a/27609):

- you need to implement `hashCode` and `equals` consistently
- it's easy to add a new field but forget to add it to one or both of these methods, causing subtle bugs
- exhaustive testing for incorrect `equals` and `hashCode` methods is rarely done

At the same time, you don't want it to be a `case class`, because:

- case classes provide more than just equality. In particular the `.copy()` method allows you to construct new instances with modifications to arbitrary fields
  - this might be unwanted, if you only want to expose a single public builder method which enforces some invariants
  - it exposes a larger api surface which you don't necessarily want to commit to, and might cause [binary compatibility issues](https://docs.scala-lang.org/overviews/core/binary-compatibility-for-library-authors.html)
- maybe it just doesn't fit with the [case class](#case-classes) guidelines

In these cases you can hide the "guts" of your object in a private case class, and delegate the desired methods to it explicitly:

```
class Person(name: String, age: Int, favouriteMovies: List[String]) {
  private val data = Person.Data(name = name, age = age, favouriteMovies = favouriteMovies)

  override def toString() = data.toString()
  override def hashcode() = data.hashCode()
  override def equals(obj: Any): Boolean = obj match {
    case obj:HealthService => impl.equals(obj.impl)
    case _ => false
  }
}

object Person {
  private [Person] case class Data(name: String, age: Int, favouriteMovies: List[String])
}
```

Nobody else will be able to see or use the case class, but its auto-generated `equals` and `hashCode` implementations have less risk of bugs than manually written ones.

Here's an example in [zendesk-consul-scala](https://github.com/zendesk/zendesk-consul-scala/blob/4f1959b4a86f14b95c72306d2ac59ed5bdaea966/src/main/scala/com/zendesk/consulclient/ConsulOp.scala#L39-L64).

## Values representing times must include the time unit in the name

Variable names for things representing a time must include the time unit in the name. This is to avoid confusion and incorrect comparisions between different time units where the information is not conveyed by the type alone (eg `Long`).

Examples:
- Data eg `case class MyObject(timestampMs: Long)`
- Metrics eg `zendesk.escape.latency_ms` (not `zendesk.escape.latency`)
- Schemas
  - protobuf
  - database columns
  - json

Exceptions:
- Use a `timeout: FiniteDuration` on configuration to represent service timeout rather than `timeoutMs: Long`
- Private implementation details where all values within a function share the same time unit
- This would be pretty painful to apply retrospectively, so only suggesting for new things

# Still Up for debate
> Topics that need more clarification or debate

## Test coverage
We'll need to discuss this again.

## The only things that should be public are those you expect to be called
- Don't make it public just for tests, then it's just coupling to internal implementation details and will be brittle if refactored.
- **TODO**: should we expose with `private [packagename] def ...` for tests
- **TODO** discuss later.


## Don't auto manage splitting code lines
- it impacts code readability
- Enforce in CI
- **TODO**: share between programs (make consistent)
- **TODO**: turn off line length

## Use of parentheses for zero argument methods
- When calling a no argument method, use brackets if it was declared with brackets (Scala 2 will let you omit them but that can be confusing)
- When declaring methods, [Scala docs](https://docs.scala-lang.org/style/method-invocation.html#arity-0) recommends omitting parentheses on methods of arity-0 **if** the method has no side-effects
- For no-argument methods that have side effects, use parentheses
- **Why?** To improve readability so that you can see at a glance the intention of the code e.g `queue.size` (which is pure) vs `println()` (which has side-effects)
