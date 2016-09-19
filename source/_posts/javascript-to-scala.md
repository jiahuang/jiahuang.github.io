---
title: A JavaScript developer's Scala cheatsheet
date: 2016-09-10 18:50:25
tags:
---

I've been working in Scala for the past year. When I first started, I was pleasantly surprised by how similar it was to JavaScript. Scala is a functional language, supports streams, and has async programming through futures/promises. Conceptually, good JS code is good Scala code. But at the same time Scala is also an object oriented, typed language that runs on the JVM.

When writing JavaScript, it's a breeze to start programming. However, I find myself having to exert a lot of mental energy closer to the end, when the program is coming together and I need to make sure the duck typing is handling all the edge cases. It's the opposite with Scala where all my effort is frontloaded in making sure all the return types match. Once that happens, it's almost trivial to write the logic.

Here's an amalgamation of things I wish I saw when first learning Scala.

**Note: The examples are written with Scala 2.11.8 & Akka version 2.4.10**

### Packaging & Tools
The Scala equivalent of npm is [sbt](http://www.scala-sbt.org/), the Simple Build Tool. Like npm, sbt handles packaging, dependencies, building, running, and testing.

Here are some commonly used sbt commands from the cli:
```sh
sbt run // runs the code
sbt run -DKey1=1 -DKey2=2 // optional variables to use when starting to run the script
sbt publish // publishes a package
sbt publish-local // publishes a package locally. Useful if you are developing multiple packages at once
sbt test // runs all tests
sbt "test:test-only TestClassName" // runs a specific test class name
```

sbt requires a `build.sbt` file in the main directory. It will look something like this:

```js
enablePlugins(JavaServerAppPackaging) // see http://www.scala-sbt.org/sbt-native-packager/archetypes/java_server/

name := "testproject" // name of your package
organization := "com.mysite" // the package name will be com.mysite.testproject for namespacing
version := "1.0"
scalaVersion := "2.11.8" // version of scala this requires

// list of package dependencies
libraryDependencies ++= {
  val akkaV = "2.4.4"
  val scalaTestV = "2.2.4"
  Seq(
    "com.typesafe.akka" %% "akka-actor" % akkaV,
    "com.typesafe.akka" %% "akka-stream" % akkaV
  )
}
```

### Basics
#### Variables
Variable declaration is similar, with the exception of Scala having `var` and `val`. `val` is a `const` in JS. In Scala, it's highly preferred to use `vals` when possible. Scala can be interpreted so to test these out just run `scala` from the cli.

```scala
val a = "a"
// however if you want to reassign a variable, it should be declared as a "var"
var a = "b"
// array
val list = List(item1, item2, item3)
// object
case class TestClass(a: Int, b: Int, c: Int)
TestClass(1, 2, 3)
```

#### Functions
Functions cannot exist alone outside of classes, traits, or objects.

```scala
// def creates a method
// followed by add which is the method name
// aa takes in 2 parameters which are both of type Int
// and the Int after the colon is the return type of the method
def add(a: Int, b: Int): Int = {
  // the result of the last line is returned by default
  a + b
}
```

Methods can take in a callback:
```scala
// callback is a function that takes in an int and returns nothing
def sleep(a: Int, callback: (Int) => Unit) = {
  Thread.sleep(a*1000)
  callback(a)
}

sleep(3, (time) => {println(s"slept for $time seconds")}) // prints "slept for 3"
```

However Futures/Promises are generally preferred. This also avoids callback hell:
```scala
def doSomething(a: String): Future[Int] = {
  val p = Promise[Int]
  // where someFunc is also a future that will take some time to complete
  someFunc(a).onComplete {
    case Success(res) => p.success(res)
    case Failure(ex: Throwable) => { p.failure(ex) }
  }
  p.future
}
```

Functions can be `flatMap`ed to convert from one future type to another future type.

```scala
// some method definitions
def someFunc(a: Int): Future[String]
def someOtherFunc(str: String): Future[Int]

def doSomething(a: Int): Future[Int] = {
  // someFunc returns a String, but doSomething needs an int
  // someOtherFunc can return an int, so flatMap can change the
  // Future[String] to a Future[Int]
  someFunc(a).flatMap{res =>
    someOtherFunc(res)
  }
}
```

#### Objects
In Scala there are classes, objects, and traits. Classes can be instantiated, objects and traits cannot. Traits can be mixed in to other classes or objects to extend a class/object's functionality. In JS, this is all handled via the prototype chain.

There are case classes and regular classes. Case classes have tons of boilerplate that makes them easier to use along with enforced immutabilitly. Regular classes should be avoided unless there's a bunch of mutable state in the class (which should be avoided anyway).

This would be the equivalent of instantiating a JS class:
```scala
case class TestClass(a: Int, b: Int) // takes in 2 parameters which are all integers. these 3 values are "vals" by default
val c = testClass(1, 2)
c.a // is 1
c.a = 5 // this will fail because you cannot reassign to a "val"

// you can have the case class take var instead if parameters will be mutable
case class TestClass(var a: Int, var b: Int)
val c = TestClass(1, 2)
c.a = 5 // this will work

// case classes can also have methods
case class TestClass(a: Int, b: Int) {
  // because this method does not need to take in a parameter
  // there is no need to have parens like isAGreater()
  def isAGreater: Boolean = {
    a > b
  }
}

val c = TestClass(1, 2)
c.isAGreater // returns false
```

Static methods can be created by using a Scala Object:
```scala
object Obj {
  def add(a: Int, b: Int): Boolean = {
    a + b
  }
}
Obj.add(1, 2) // returns 3
```

Instead of the prototype chain, traits can be used to mix in methods
```scala
trait Animal {
  val noise: String
  def makeNoise = {
    println(noise)
  }
}

case class Dog(noise: String) extends Animal
val spot = Dog("ruff")
spot.makeNoise
```

#### Array functions
Array functions work similarly in JS & Scala. Both languages have array functions that can be chained and take in lambda functions.

```scala
var list = List(1, 2, 3, 4)
list.map{f => f+1}
println(list) // prints 2, 3, 4, 5
// a shorthand is to use _
list.map{_+1}
```

### Streams
I heavily used the Akka library which includes [Akka-streams](http://doc.akka.io/docs/akka/2.4.10/scala/stream/index.html). I found that Scala streams are way more complicated and harder to understand compared to JS streams. JS streams have an emitting function that can be listened to as chunks of data are read. It is then up to the listening function to handle each chunk of data correctly.

In Scala the read stream is a `source`. The source connects to a `sink`. In between the source and the sink can be several processing stages. Each of these processing stages can be abstracted out into a `flow`. Once the source and sink are connected the stream can be run.

Here's an example that creates a source of numbers and filters out anything starting with 1:
```scala
val source: Source[Int, NotUsed] = Source(1 to 100)
source
  .filter{i => i.toString.map(_.asDigit).head > 1}
  .runForeach(println)

// can be rewritten to make the filter step a flow
val filterOnes = Flow[Int].filter{i => i.toString.map(_.asDigit).head > 1}
val sink = Sink.foreach[Int](println)
val runnable = source via filterOnes to sink
runnable.run()
```

Flows are used for linear processing. There are also [`graphs`]() for branched processing.

### Actors
The Akka library also includes [Akka Actors](http://doc.akka.io/docs/akka/2.4.10/scala/actors.html). Actors encapsulate state & behavior and communicate to other actors via a messaging inbox which allow actors to act asynchronously. Messages in the inbox are handled in the order that they are received so the actor never gets overscheduled for tasks.

Here is a simple actor that logs what it receives:
```scala
import akka.actor.Actor
import akka.actor.Props

// haveing a sealed trait for all messages of a particular type makes it easier to
// match on the message types later
sealed trait ActorMessages

case class Message(msg: String, importance: Int) extends ActorMessages

class Echo extends Actor {
  def receive = {
    case Message(msg: String, importance: Int) =>
      println(s"got $msg")
  }
}
```

Actors can be created by other actors. Supervising actors can also handle error codes thrown by children through a [supervisor strategy](http://doc.akka.io/docs/akka/current/scala/fault-tolerance.html).

```scala
sealed trait ActorMessages
case class CreateChild(name: String) extends ActorMessages
case class DoSomething(action: String) extends ActorMessages

class Parent extends Actor {
  // a supervisor strategy handles what happens when child actors fail
  override val supervisorStrategy = OneForOneStrategy(maxNrOfRetries = 3, withinTimeRange = 1 minute) {
    case ex: Exception =>
      println("child actor failed", ex.printStackTrace)
      Restart // this will restart the failed child actor
  }

  def receive = {
    case CreateChild(name: String) =>
      // create a new child actor
      val child = context.actorOf(Props(new Child(name)))
      // send a message to the created child
      child ! DoSomething("throw error")
  }
}

class Child(name: String) extends Actor {
  def receive = {
    case DoSomething(action: String) =>
      action match {
        case "throw error" => throw new RuntimeException("child actor throwing error")
        case _ => println("doing", action)
      }
  }
}
```

### Programming in Scala
Having to work with Scala in the past year has definitely made me a better programmer. It has made me formalize the way I approach a programming problem. The duck typing in JavaScript makes it all too tempting for me to bolt on features as I go, and end up with an unholy mess at the end. Scala also exposed me to other ways of looking at streams and concurrency, and gave me some new mental models for solving problems. 
