---
layout: post
title: Top and Bottom
---

As a follow-up to [my previous post on `Unit` vs `null`]({% post_url 2018-09-05-unit-vs-null %}), I thought it might be useful to talk about some of the other special types we have in statically typed languages.

There are actually two other special types in the type-system that you may be interested in: namely, the bottom and the top type.

## The Bottom Type

The bottom type (written as `Nothing` in Scala) is a type that is impossible to create. It is commonly used as a placeholder when a type isn't known to the compiler. See for example what happens when we don't tell the compiler the types of the keys and values in an empty `Map`:

```scala
scala> Map.empty
res0: scala.collection.immutable.Map[Nothing,Nothing] = Map()
```

As it has no other information to go on, the types for both end up as `Nothing`.

It's also useful for saying that a function never returns — for example, a function that exits the program before returning could be written as:

```scala
def doSomething(): Nothing = sys.exit(1)
```

It should be clear from the type that it is impossible for this function to produce a return value. In fact, it's not even possible to write a function that has a real return value if you have `Nothing` as the return types:

```scala
scala> def doSomething(): Nothing = 123
<console>:11: error: type mismatch;
 found   : Int(123)
 required: Nothing
       def doSomething(): Nothing = 123
```

 You therefore know that upon seeing a function returning `Nothing`, it will _definitely_ never return if you call it — all guranteed because it's impossible to make a value of type `Nothing`!

## The Top Type

The opposite of the bottom type is the top type. Also called the universal type (and written in Scala as `Any`), this is simply a way to say "this could be any possible type". For example:

```scala
def doSomething(): Any = 123
```

This function actually returns an `Int` but typing it `Any` works — what's going on here? This is because `Any` is actually the ancestor of _all_ types in Scala, even `Object`:

```scala
scala> val x: Object = new Object {}
x: Object = $anon$1@b428830

scala> x.isInstanceOf[Any]
res1: Boolean = true
```

 You'll often see `Any` used by the compiler when it is trying to fill in a missing type (so called "type ascription") but couldn't find anything more specific to choose.

## Why Do We Need These Types?

The reason both bottom and top types have to exist comes down to how static-typing works — compilers of statically typed languages are, fundamentally, trying to solve two problems:

* Type-ascription: That is, filling in missing types by finding the most specific type that something should be.
* Constraint solving: Proving correct or finding a set of constraints on types given a heirachy of types. This is used for things like subtyping and supertyping, and is called "variance".

The bottom type is a natural way to represent an impossible scenario during the first of these tasks. The bottom type is the default type if a type cannot be ascripted — that is, when you don't annotate a type, `Nothing` will be the end result if the compiler cannot find another more specific type after searching through everything else.

The top type is the default for constraint reduction. It gives a compiler a final ultimate result if nothing can be proven when doing constraint reduction. `Any` is what you end up with when two types constrain each other so much that they have no other ancestor in common, and works because _every single type_ has `Any` at least in common.
