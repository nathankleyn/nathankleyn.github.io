---
layout: post
title: Using The Lazy Enumerator In Ruby
---

Recently, I've found myself starting to use the new Ruby 2.0 lazy enumerator feature quite a bit. I'd [written about the lazy enumerator feature in the past](http://www.sitepoint.com/functional-programming-techniques-with-ruby-part-iii/), but had previously never found much time to use it, what with backwards incompatibilities and all making it difficult to use this in production code.

I'm finding there are two main circumstances in which I'm using this feature:

* **Speeding up code where multiple iterations of the dataset is unavoidable when using strict enumerations.** This is the obvious "poster-child" use case for this feature, whereby multiple iterations can be magically optimised into a single iteration.
* **Cleaning up code that does a lot in a single iteration.** This somewhat surprised me, as I always thought that Ruby's enumerator syntax was about as good gets, or at least certainly compared to most other languages.

## Speeding up multiple iterations

In versions of Ruby without the lazy enumerator, sometimes you have to suffer the penalty of multiple iterations because you need to use multiple enumerator methods:

{% highlight ruby %}
def do_some_stuff
  some_stuff
    .flat_map { |x| do_something(x) }
    .sort_by(&:something)
    .reverse
    .uniq(&:something_else)
end
{% endhighlight %}

With a two line sprinkle, you can dramatically change the performance characteristics of this code:

{% highlight ruby %}
def do_some_stuff
  some_stuff
    .lazy
    .flat_map { |x| do_something(x) }
    .sort_by(&:something)
    .reverse
    .uniq(&:something_else)
    .to_a
end
{% endhighlight %}

Now you'll only suffer a single iteration of `some_stuff`, which can add up to a massive saving if that enumerable object ever gets large! Obviously, the laziness does add a performance penalty, so you need to exercise caution and add the lazy enumeration only when the number of iterations justifies its usage.

## Cleaning up code large enumeration blocks

I often find myself wanting to refactor code that does a lot in a single enumeration block, but often refactoring this out to methods can be overkill. Take for example this (extremely contrived) example:

{% highlight ruby %}
xs.select { |x| x % 3 == 0 && x % 4 == 0 && x.to_s.size < 5 }
{% endhighlight %}

You can end up with this pretty gross "one block to rule them all" style code easily by simply trying to avoid the cost of iterating. This code is but a small taste of the size of some blocks I've seen out there, and it can make code hard to read when you try to make one block do too much work.

Enter the lazy enumerator:

{% highlight ruby %}
xs.lazy
  .select { |x| x % 3 == 0 } # Nums divisible by 4
  .select { |x| x % 4 == 0 } # Nums also divisible by 3
  .select { |x| x.to_s.size < 5 } # Nums less than 5 digits long
  .to_a
{% endhighlight %}

This gives you an opportunity to refactor your code to do a single thing per line, and do it tersely. You also have the ability to be able to comment each line as you need without introducing local variables, leading to cleaner and more readable code.

## Learnings

In any code where you can use Ruby 2.0+, consider using the lazy enumerable for these use cases. You may find it leads to cleaner, more readable code - and it some cases a hefty performance gain!
