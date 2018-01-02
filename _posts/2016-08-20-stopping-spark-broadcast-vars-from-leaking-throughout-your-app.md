---
layout: post
title: Stopping Spark Broadcast Variables From Leaking Throughout Your Application
---

When working with Spark it's tempting to use `Broadcast` variables everywhere you need them without thinking too much about it. However, if you are not careful, this can lead to strong coupling between all of your application components and the implementation of Spark itself.

### How Broadcast variables work

[Broadcast variables](https://spark.apache.org/docs/latest/programming-guide.html#broadcast-variables) are [intended to solve a very specific problem in Spark](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-broadcast.html): namely, preparing some static value on the driver side to be passed efficiently to all nodes that need it during some processing step. Spark takes care of implementing the distribution of this variable in the most efficient manner, eg. it can elect to use BitTorrent for larger variables (if enabled), and promises that only the nodes that need the data will be sent it at the point they need it.

However, in order to work, broadcast variables effectively provide a wrapper type for the variable they close over:

```scala
// This is a vastly simplified version of the class!
abstract class Broadcast[T: ClassTag](val id: Long) {
  // ...
  def value: T = // ...
  // ...
}
```

Effectively, Spark will store our data and reference it via an ID (a `Long`) which is all we need to send to the nodes until they actually need the data. When they need the data, they send the driver the id, and they get what they need back!

### Working with Broadcast variables

In order for us to setup and get the value out of a broadcast variable, we need to do something like the following:

```scala
val data: Broadcast[Array[Int]] = sparkContext.broadcast(Array(1, 2, 3))

// Later on, to use it somewhere...
data.value // => Array[Int] = Array(1, 2, 3))
```

### Leaking the types

When working with Spark, it can be easy to end up with one "god job" where the implementation of all the steps of a job are inlined and heavily dependent on Spark features.

With some effort, we can factor out our pure functions into smaller classes which we can call into:

```scala
class PostcodeToLatLong(lookup: Map[String, (Double, Double)]) {
  def lookupPostcode(postcode: String): (Double, Double) = ???
}
```

Here we have a simple class which needs a lookup table to function: in this case, it will convert some string based postcode to a latitude/longitude pair which is represented as a tuple of `Doubles`.

If this class was going to be used in a Spark job, we may consider making the `postcodeTable` a broadcast variable so that only the nodes that need it will request the data for it. However, we hit a snag: we can't do this without leaking the `Brodcast` wrapper type into the implementation details of the class like so:

```scala
class PostcodeToLatLong(lookup: Broadcast[Map[String, (Double, Double)]]) {
  // ...
}
```

This is a nightmare for testing, as now we've gone from having a simple, pure class which can be unit tested easily to having a class that depends entirely on Spark implementation details and needs a `SparkContext` just to setup!

### Abstracting away the broadcast functionality

Thankfully, we can solve this using a trait that abstracts what we actually care about: that the thing we have passed is lazy and will only be grabbed when we call `.value` on it!

```scala
trait Lazy[T] extends Serializable {
  def value: T
}
```

And now we can change our class to take this trait instead:

```scala
class PostcodeToLatLong(lookup: Lazy[Map[String, (Double, Double)]]) {
  // ...
}
```

With some simple implicit classes we can make it easy to call this class with either a Spark broadcast variable or any old primitive object:

```scala
object Lazy {
  object implicits {
    implicit class LazySparkBroadcast[T: ClassTag](bc: Broadcast[T]) extends Lazy[T] {
      override def value: T = bc.value
    }

    implicit class LazyPrimitive[T: ClassTag](inner: T) extends Lazy[T] {
      override def value: T = inner
    }
  }
}
```

And now we can put the pieces together:

```scala
import Lazy.implicits._

val lookup = Map("foo" -> (123.45, 123.45))
val bcLookup = sparkContext.broadcast(lookup)

// And later on...

val postcodeMapper = new PostcodeToLatLong(bcLookup)
// This also works!
val postcodeMapper = new PostcodeToLatLong(lookup)
```
