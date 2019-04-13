---
layout: post
title: Homebrew >= v1.9.0 No Longer Links MacOS Provided Software
---

I recently had a bit of fun after upgrading to Homebrew `v1.9.0`, so wanted to write a quick PSA for others who may hit the same issue.

[Homebrew >= `v1.9.0`](https://brew.sh/2019/01/09/homebrew-1.9.0/) has a [breaking change to `brew link --force`](https://github.com/Homebrew/brew/pull/5383):

> `brew link --force` will not link software already provided by macOS.

That is, this change means that Homebrew will no longer allow `brew link` to override anything MacOS already ships with.

So for example, if you used Homebrew to install a recent version of Ruby, before `v1.9.0` Homebrew's version would've been in your path — now the MacOS system version will be in your path:

 ```sh
$ ruby -v
ruby 2.3.7p456 (2018-03-28 revision 63024) [universal.x86_64-darwin18]
$ /usr/local/opt/ruby/bin/ruby -v
ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin18]
```

If you try to `brew link` it, Homebrew will tell you why it's refusing to, and give you a snippet of script for prepending it to your `PATH` in your chosen shell (in my case, `fish`):

 ```sh
 $ brew link --force ruby
Warning: Refusing to link macOS-provided software: ruby
If you need to have ruby first in your PATH run:
  echo 'set -g fish_user_paths "/usr/local/opt/ruby/bin" $fish_user_paths' >> ~/.config/fish/config.fish

For compilers to find ruby you may need to set:
  set -gx LDFLAGS "-L/usr/local/opt/ruby/lib"
  set -gx CPPFLAGS "-I/usr/local/opt/ruby/include"

For pkg-config to find ruby you may need to set:
  set -gx PKG_CONFIG_PATH "/usr/local/opt/ruby/lib/pkgconfig"
```

You can also see this information for any given package by running `brew info <package>`.

The upgrade to `v1.9.0` does not give any warning about this — the change is effectively silent — so beware of scripts depending any binaries that clash with MacOS provided ones being in your default system `PATH`!
