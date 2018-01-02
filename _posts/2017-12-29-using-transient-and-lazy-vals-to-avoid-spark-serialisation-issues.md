---
layout: post
title: Using Transient Lazy Val's To Avoid Spark Serialisation Issues
---

Occasionally within a Spark application, you may encounter issues because some member of a (`case`) `class` / `object` is not serialisable. This manifests most often as an exception when the first task containing that item is attempted to be sent to an executor. The vast majority of the time, the fix you probably will reach for is to make the object implement `Serializable`. Sometimes however, this may not be easy or even possible (for example, when the type in question is out of your control).

It turns out that there is another way! If the object in question can be constructed again inexpensively, the `@transient lazy val` pattern may be for you:

```scala
case class Foo(foo: SomeType) {
  @transient lazy val bar: SomeOtherType = SomeOtherType(foo)
}
```

The `@transient` annotation has the effect of excluding the annotated item from the object it is contained within when that object is serialised. In conjunction with the `lazy val`, this means the field will be constructed again when first accessed on each of the executors, rather than being sent to each executor as a series of serialised bytes to deserialise as part of the task.

Sometimes this trick can actually result in modest performance improvements — for example, if the object is question is large when serialised but is cheap to construct again (like large collections computed from some smaller seed). However, carefully note that it is constructed again _once per executor_ making it only useful for stateless items.

The next time you hit a serialisation issue in Spark, give `@transient lazy val` a try and see if it's a fit for you!
