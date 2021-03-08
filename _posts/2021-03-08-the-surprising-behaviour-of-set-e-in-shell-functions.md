---
layout: post
title: The Surpring Behaviour Of `set -e` In Shell Functions
---

Today I just learned something about shell/bash that I never knew before which caused me quite a bit of confusion.

If you have `set -e` on in a bash/shell script, as you probably know the shell will exit if any line has a non-zero exit status and it isn’t caught and handled by something.

But what I did not know before today was that this is "weird" inside functions:

```sh
set -e

foo () {
    echo "I should be printed first!"
    false
    echo "I should not be printed?"
}

foo || echo "I should be printed last!"
```

Maybe somewhat surprisingly the output of this is:

```text
I should be printed first!
I should not be printed?
```

This happens because **if you handle the output of an entire function then it kind of acts like every line inside the function is also handled**. This means lines can run that you were not expecting!

Additionally, **the last line that runs will be the exit status of the function**, so in this case being `echo` my "I should be printed last!" line didn’t even run because `echo` always returns a zero exit status!

Therefore, it’s more correct to do:

```sh
set -eo pipefail

foo () {
    echo "I should be printed first!"
    false || echo "I should be printed last!" && return 1
    echo "I should not be printed?"
}

foo
```

That is, **handle each line failing where it happens, and don’t try to handle whole functions**.

This [is documented in _some_ of the various versions of documentation for `set -e`](https://www.enseignement.polytechnique.fr/informatique/INF422/sh.html):

> `-e` errexit
>
> Exit immediately if any untested command fails in non-interactive mode. The exit status of a command is considered to be explicitly tested if the command is part of the list used to control an if, elif, while, or until; if the command is the left hand operand of an "`&&`" or "`||`" operator; or if the command is a pipeline preceded by the `!` operator. **If a shell function is executed and its exit status is explicitly tested, all commands of the function are considered to be tested as well.**

But I think this is almost certainly surprising behaviour for most users of shell.
