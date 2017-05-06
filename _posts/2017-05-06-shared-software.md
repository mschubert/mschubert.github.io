---
layout: post
title: Shared HPC software installations
---

I work at a research institute with considerable computing capabilities. Things
you can't run on your normal work machine, you send to a high performance
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
makes sense because everything needed to be stable. That meant the system
libraries are generally out of date by a couple of years.

We were lucky to have a person who, next to his regular responsibilities
installed software that people needed on a semi-regular basis. Or people
compiled different tools themselves. That resulted in a system with a `$PATH`
that contained too many directories and that had to be constantly amended. And
`$LD_LIBRARY_PATH` too, of course, which resulted in broken dependencies all
the time because something didn't quite resolve right.

Manually compiling things of course often resulted in disabling optional
functionality in order to get it done quickly (BLAS/LAPACK support for R? Then
I'd need to install two more libraries). And it took a substantial amount of
time to get anything running.

### Package managers to the rescue

The obvious answer to the above mess is to use a package manager to take care
of dependencies automatically and provide a much larger set of available
software or build scripts to draw from than was readily available on the
system.

At the time (first half of 2014), however, there were not many options
available. After a long search, I discovered the [Gentoo Prefix
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
people relied on it, and it worked really well for two years to come (with
minor hiccups on occasion).

### Share instructions, not installations

Since then, however, things have evolved. Linuxbrew (the linux-pendant to the
OS-X package manager Homebrew) and Conda were released. And they were
substantially simpler to use than Gentoo.



### Alternatives

docker: same issues as first
and making the same mistakes that PMs have fixed a long time ago
