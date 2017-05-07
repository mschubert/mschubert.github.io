---
layout: post
title: Shared HPC software installations
---

*TL;DR: Keep your own installation on a computing cluster, don't rely on
others.  But do use a package manager. And
[share](https://github.com/Linuxbrew/brew/blob/master/CONTRIBUTING.md) the
build scripts with the upstream repository, so everyone can easily maintain and
extend their own tool collection.*

I work at a research institute with considerable computing capabilities. Things
you can't run on your normal work machine you send to a high performance
computing cluster. Administration is taking care of by a group of people
specifically employed to make sure things run smoothly, so you just log in and
run whatever computing task you require.

Well, almost.

Of course, there is some software needed in order to run your analyses. You
installed this on your local machine a long time ago using your OS's package
manger. But is it available on the computing cluster?

### Typical HPC setup

The system administrators were happy to add software available in the official
distributions' repository but not from third party sources. Of course, this
makes sense because everything needed to be stable. And stable meant the system
libraries are generally out of date by a couple of years.

We were lucky to have a person who, next to his regular responsibilities
installed software that people needed on a semi-regular basis. Or people
compiled different tools themselves. Every new tool was a new entry in `$PATH`,
which grew continuously. And `$LD_LIBRARY_PATH` too, of course, which
resulted in broken dependencies all the time because something didn't
quite resolve right.

Manually compiling things often resulted in disabling optional functionality in
order to get it done quickly (BLAS/LAPACK support for R? Then I'd need to
install two more libraries). And it took a substantial amount of time
to get anything running.

### Package managers to the rescue

The obvious answer to the above mess is to use a package manager to take care
of dependencies automatically and provide a much larger set of available
software or build scripts to draw from than was readily available on the
system.

At the time (first half of 2014), however, there were not many options
available. After looking for possible solutions for a couple of weeks I
discovered the [Gentoo Prefix
project](https://wiki.gentoo.org/wiki/Project:Prefix) that provided a
user-level package manager which could use (almost) all the build scripts from
the distribution.

This was great! It provided a base installation that enabled a non-root user
the use of [portage](https://en.wikipedia.org/wiki/Portage_(software)) in a
specified directory, resolved dependencies automatically, and resolved library
locations using `RPATH` instead of the `$LD_LIBRARY_PATH` mess.

{% highlight sh %}
emerge <whatever you want>
{% endhighlight sh %}

Magic.

Only downside was that the bootstrap and compilation options were a bit
involved, so it probably wasn't for everyone to take care of this. Hence [I set
this up at my institute](https://github.com/EBI-predocs/research-software) and
advocated that people use it.  We had a [bug
tracker](https://github.com/EBI-predocs/research-software/issues), [software
requests](https://github.com/EBI-predocs/research-software/issues?q=is%3Aissue+is%3Aclosed+label%3A%22software+request%22),
and [advertised
updates](https://github.com/EBI-predocs/research-software/issues?q=is%3Aissue+is%3Aclosed+label%3Aannouncement)
before we performed them so users are not surprised by small changes. About 30
people relied on it, and it worked really well for three years to come.

### Don't use shared software installations

We had [minor
hiccups](https://github.com/EBI-predocs/research-software/issues?q=is%3Aissue+is%3Aclosed+label%3Abug)
on occasion and a [major
hiccup](https://github.com/EBI-predocs/research-software/issues/52) once
(basically, the linker added symbols from a broken internationalization library
 that cascaded into breaking everything newly added or updated). But these were
a small price to pay for a usable and convenient system.

One thing that was annoying is that libraries and tools sometimes change
API between updates, and that would break a workflow that you are
currently using. But since this was shared, a change for anyone was a change for
everyone.

Since then, however, things have evolved. [Linuxbrew](http://linuxbrew.sh/)
(the linux-pendant to the OS-X package manager [Homebrew](https://brew.sh/))
and [Conda](https://conda.io/docs/) were released. And they were substantially
simpler to use than Gentoo.

So I don't really see a reason to still put up with API changes for any user
unless they want to update their installation. The new package managers are
easy enough to use individually, and storage space is cheap so why bother.

### But please contribute your build scripts

There is, however, one thing to keep in mind that often goes underappreciated.
If you need a tool that doesn't yet exist in a repository and you already put
the effort in to manually install it, go one step further and add is as a build
script for the package manager. Please. It helps so much if more people do that.
