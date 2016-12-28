---
layout: post
title: A sensible .Rprofile
---

Here are some settings that I found useful for working with R. Note that I'm
mainly using basic R scripts in an HPC environment and not
[Rstudio](https://www.rstudio.com/) most of the time.

TL;DR: have a look at [the whole file](https://gist.github.com/mschubert/0ce46ffc599358c9c799487898860395).

### Custom library path

R is not hard to install, but there is a couple of compile-time configuration
steps to take care of:

 * Graphics backend support using [cairo](http://stackoverflow.com/q/16619746)
 * [BLAS
 support](http://brettklamer.com/diversions/statistical/faster-blas-in-r/)
 to speed up linear algebra
 * Profiling needs to be enabled in the `configure` options (this is default)

For these reasons, it will make sense to have a central install on an HPC
cluster to get the details right. However, that comes with some challenges.

In particular, the separation of user-readable system packages and
user-writable personal packages [does not always
work](https://github.com/EBI-predocs/research-software/issues/57).

This is why I found it easiest to specify an explicit library path.

{% highlight r %}
.libPaths("~/.R")
{% endhighlight %}

This way, R will ignore the system-provided packages and install all required
packages in the above directory where you have got full write access.

### Custom options

Here are some tweaks about converting strings to factors, selecting a default
CRAN mirror, and warn about partial matching of subsetting and function
arguments.

{% highlight r %}
options(
    stringsAsFactors = FALSE,

    menu.graphics = FALSE,
    repos = c(CRAN="http://mirrors.ebi.ac.uk/CRAN/"),

    warnPartialMatchAttr = TRUE,
    warnPartialMatchDollar = TRUE
)
{% endhighlight %}

There is some controversy about how R is supposed to handle strings. Convert
characters to factors by default or not? I've been a strong proponent to
disabling converting, for which I've been accused [to not understand how they
work](http://stackoverflow.com/q/26060476). True, code is less portable by
setting a non-default option, but factors mess up enough that I don't want to
deal with it.

Another tweak is to not display the graphical menu to select a mirror when
downloading from [CRAN](https://cran.r-project.org/) (this just delays what I
actually wanted to do) and select a default mirror.

A third is to dispaly a warning when I either try to pass a named function
argument that does not exactly match what the function is expecting (helps not
mixing up arguments with confusing errors after) or when subsetting a `list` or
`data.frame` using the `$` sign.

### Traceback on non-interactive scripts

When scripts run non-interactively and they fail, it is often challenging to
find the source of the error or the function call in which R failed, as only
the error message is reported and no additional context by default.

Using the below lines, you can get the traceback on errors when scripts fail
that will not impact your interactive sessions.

{% highlight r %}
if (!interactive())
    options(error = function() {
        traceback(2, max.lines=10)
        quit(save="no", status=1)
    })
{% endhighlight %}

The guard to exit R after printing the traceback is essential, because
otherwise it will happily try to execute the rest of the script with an
undefined state.
