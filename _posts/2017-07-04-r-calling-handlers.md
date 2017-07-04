---
layout: post
title: Custom handling of errors and warnings in R function calls
---

Let's consider the following function in R, and assume we want to catch its
erros and warnings for later processing:

{% highlight r %}
f = function(a, ...) {
    if (a %% 3 == 0)
        warning("this is a warning")
    if (a %% 2 == 0)
        stop("this is an error")
    a
}
{% endhighlight r %}

It's pretty simple to catch the errors, or warnings:

{% highlight r %}
# this will return a "try-error" class if an error occurred
# however, we can't handle warnings
try(f(2))

# this will catch warning messages and process them
# however, it will not continue the function execution after,
#   i.e. result is NULL
result = tryCatch(f(3),
                  error=function(w) message("process error"),
                  warning=function(w) message("process warning"))
{% endhighlight r %}

Also, if we want to process errors *and* warnings the `tryCatch()` block
only processes the warning:

{% highlight r %}
# no result, no error processing
result = tryCatch(f(6),
    error=function(w) message("process error"),
    warning=function(w) message("process warning"))
{% endhighlight r %}

If we want it to handle warnings and errors equally, as well as still the the
function result when no error occurred, we need a more complicated construct:

{% highlight r %}
f = function(...) {
    context = new.env()

    context$result = withCallingHandlers(
        withRestarts(
            fx(...),
            muffleStop = function() NULL
        ),
        warning = function(w) {
            context$warnings = c(context$warnings,
                 list(as.character(conditionMessage(w))))
            invokeRestart("muffleWarning")
        },
        error = function(e) {
            context$error = as.character(conditionMessage(e))
            invokeRestart("muffleStop")
        }
    )

    as.list(context)
}

fwrap(a=1)
fwrap(a=3)
fwrap(a=2)
fwrap(a=6)
{% endhighlight r %}
