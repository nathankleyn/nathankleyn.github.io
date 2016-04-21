---
layout: post
title: The Usefulness Of Sum Types
---

One of the great things about dynamic languages is the fact that one has duck typing at their disposal.

Unboxed union types.
I want to be able to say `Int | String` without having to box the types. This is useful in scenarios where I don't know the type up front. Basically, compiler checked overloading with enforced fallback.

Show Rust example
Show Haskell example

Show how it can be achieved in Scala

Explain how Miles Sabin achieved it.
Explain why that's not easily usable to solve the expression problem (because the types can't be extended arbitrarily).
