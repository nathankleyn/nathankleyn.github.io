---
layout: post
title: Unit vs Null
---

The other day, I was asked by somebody why we had `Unit` in Scala, and what was the difference over just using `null`. I thought this was a really interesting question — so here's my response in a slightly longer form!

## The Boolean Type

Let's start from something we know well — the Boolean type! When writing code, we use these all the time. As you know, they can only have two possible values — `true` or `false`.

The boolean type is _the smallest possible type with which we can represent information_. It can show something was `true`, or `false`, on or off, yes or no, 1 or 0 — just a single bit is all it has to work with, but it's enough [^1].

Imagine the following example function:

```scala
def doSomething(bool: Boolean): Boolean = ???
```

We have a function that takes a `Boolean` and returns a `Boolean`. We know immediately some things about this function:

* It is going to take either a `true` or `false` value.
* It is going to return either a `true` or `false` value.
* It cannot take or return any other values.

We know all of these things because the number of possible values for `Boolean` is limited. This limitation is what allows us to _reason about what a function can do just from it's type_ — and this is an incredibly useful tool that static-typing gives you.

## Parameters and Return Values As Tuples

Remember that function we wrote before?

```scala
def doSomething(bool: Boolean): Boolean = ???
```

We can think about functions in a slightly different way: we can think about their parameters being passed as tuples and them returning tuples:

```scala
// Is the same thing as:
val doSomething: (Boolean) => (Boolean) = ???
//               ^ tuple      ^ tuple
```

This is how a compiler _actually_ sees the functions you wrote — names given to parameters and syntax around showing the return type is just sugar that makes it read better to our human eyes. Really, behind the scenes, it's more like passing tuples of arguments and receiving tuples of results back.

Keep this tuple thing in mind!

## The Unit Type

Sometimes we do things where we don't care about or can't get a result — things that may have side-effects, for example. Often we need a way to say "this function returns nothing useful" — how can we do that without wasting space on a boolean if a boolean is the smallest piece of information we have to work with?

Well, it turns out there _is_ a type for saying "I have nothing useful to return" — and that type is called `Unit`.

`Unit` is a type that only has a single value (hence the name). This single value is also called "unit" but to differentiate the value from the type, in some languages we write it as `()`.

Notice something? That looks like a tuple! And indeed, that's not a mistake — `Unit` is just a way of saying "an empty tuple"!

`Unit` is useful because of an important property about the empty tuple
— there is only one possible value of it, `()`. Where, for example, `(Boolean)` could become `(true)` or `(false)`, `Unit` will always be `()` only. As a result of only having a single value, this means it is entirely useless for actually encoding any information — after all, what information can you impart if you're only allowed to respond with one thing?

Being able to say "I have no information" is _still_ a useful thing, and so we use `Unit` and `()` to do just that:

```scala
def doSomething(file: File): Unit = ???
```

Just from looking at the definition of this function, and as you were able to with the boolean example, you can tell this function _must_ have some side-effect [^2]. We could even use this information to guess what this function does (perhaps it deletes the file, or maybe calls `touch`?)

## Null: The Odd One Out

So now to the question at hand: what makes `Unit` useful over `null`?

The issue with `null` is that it is, basically, a hack! `null` is a special value that can take the place of _any type_, but it is not a type itself. This completely breaks our ability to reason using just types. Take for example this function:

```scala
def doSomethingpath: Path): File = ???
```

We might look at this and think "oh cool, maybe it's opening a `File` for us?". Well, if we didn't have `null` you might possibly be right[^3] — but unfortunately it's completely valid for this function to be implemented thusly:

```scala
def doSomethingpath: Path): File = null
```

What's wrong about `null` is that it is a magic value not represented in the type-system, and therefore it limits a lot of our ability to use the types to their full power. This is why most idiomatic Scala eschews the use of `null` over things like `Unit`, `Option[A]`, or `Either[A, B]` — these types tell the full story of what is going on, without hacks, and allow us the full power of deductive reasoning. `null` in Scala exists purely because of Java compatability, and you would be wise to avoid it at all costs!

<!-- ## Extra: Top and Bottom

There are actually two other special types in the type-system that you may be interested in: namely, the bottom and the top type.

The bottom type (written as `Nothing` in Scala) is a type that is impossible to create. It is commonly used as a placeholder when a type isn't known to the compiler. It's also useful for saying that a function never returns — for example, a function that exits the program before returning could be written as:

```scala
def doSomething(): Nothing = sys.exit(1)
```

This makes it clear from the type that it is impossible for this function to produce a return value and it _will_ never return if you call it — all guranteed because it's impossible to make a value of type `Nothing`!

The opposite of the bottom type is the top type. Also called the universal type (and written in Scala as `Any`), this is simply a way to say "this could be any possible type". For example:

```scala
def doSomething(x: Any): Any = x
```

This is a completely valid implementation of `identity` but without using a type parameter. You'll often see `Any` again used by the compiler when it is trying to ascript a type and it is seeing a lot of types that have nothing in common — this is because `Any` is the anscetor of _all_ types in Scala, even `Object`:

```scala
scala> val x: Object = new Object {}
x: Object = $anon$1@b428830

scala> x.isInstanceOf[Any]
res1: Boolean = true
``` -->

<!-- ## Extra: Bottom and Top

### The Bottom Type

In type-theory, we often want a way to express something that is said to be "incalculable"

### The Top Type

The opposite of the bottom type is the "top type". This

* Zero type (bottom) (`Nothing`)
* Unit type
** Similarity to Void
* Null
* Any () -->

[^1]: Space is an important limitation in computers, and it is natural for us to seek out the smallest way to represent things — booleans are as small as it gets. It also fundamentally encodes the entirety of boolean logic, which enables us to write code that can make decisions based on conditional statements.
[^2]: Assuming you aren't making empty functions, of course.
[^3]: Techically, exceptions are also another thing that isn't represented in the type-system. Languages like Rust are taking this thinking to the mainstream and using an `Either` like type (`Result`) and `Option` completely instead of exception!
