---
layout: post
title: Making Use Of The Lazy Enumerator In Ruby
---

Recently, I've found myself starting to use the new Ruby 2.0 Lazy Enumerator feature quite a bit. I've [written about the lazy enumerator feature in the past](http://www.sitepoint.com/functional-programming-techniques-with-ruby-part-iii/), but never found much time to use it what with backwards incompatibilities and all.

I'm finding there are two main circumstances in which you'll want to use this feature if your use case allows Ruby 2.0+:

* Speeding up code where multiple iterations of the dataset is unavoidable with strict enumerations. This is the obvious "poster-child" use case for this feature.
* Cleaning up code that does a lot in a single iteration. This is somewhat surprising to me, as I always thought up until this point that Ruby's enumerator syntax was as good as it could get compared to most other languages.

## Speeding up multiple iterations

In versions of Ruby without the lazy enumerator, sometimes you had to suffer the penalty of multiple iterations because you needed to use multiple enumerator methods:

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

Now you'll only suffer a single iteration of `some_stuff`, which can add up to a massive saving if that enumerable object ever gets large!

## Cleaning up code large enumeration blocks

Sometimes I want to refactor code that does a lot in a single loop into something that reads better. Take for example this (extremely contrived) sample code:

{% highlight ruby %}
xs.select { |x| x % 3 == 0 && x % 4 == 0 && x.to_s.size < 5 }
{% endhighlight %}

Often one can end up with this pretty gross method nesting just because you want to avoid the cost of iterating. This code is likely but a small taste of the size of some blocks out there, and it can make code hard to read when you try to shove too much in a single block.

Enter the lazy enumerator:

{% highlight ruby %}
xs.lazy
  .select { |x| x % 3 == 0 } # Nums divisible by 4
  .select { |x| x % 4 == 0 } # Nums also divisble by 3
  .select { |x| x.to_s.size < 5 } # Nums less than 5 digits long
  .to_a
{% endhighlight %}

This gives you an opportunity to refactor your code to do a single thing per line and do it tersely. You also have the ability to be able to comment each line as you need, leading to cleaner, more readable code.

## Learnings

In any code where you can use Ruby 2.0+, consider using the lazy enumerable for these use cases. You may find it leads to cleaner, more readable code - and it some cases a hefty performance gain!
