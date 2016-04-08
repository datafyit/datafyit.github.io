---
layout: post
author: Uroš Trebec
title:  "TIL: Dynamic variable names in Bash"
date:   2016-04-08 19:13:00 +0100
categories:
    - til
tags:
    - docker
    - docker-compose
    - bash
---

From time to time one need to do some Bash scripting. Lately I've come back to this while writing a Docker entrypoint for a new service we're launching.

Docker entrypoint scripts are usualy where you try to mangle together some environment variables into other environment variables to make use of in the application.

Today I learned how to dynamicaly create a variable and then use it to fetch a value from existing (environment) variable.

{% highlight bash %}
#!/bin/bash
set -e

# Using this for the dynamic part
COUNT=1

# The VALUE must to be set from an existing variable, but don't know exactly how its named
until [ -n "$VALUE" ]; do
    # Construct a dynamic variable name
    NAME="SOME_DYNAMYCALY_NAMED_VARIABLE_${COUNT}"

    # Let me know what's going on
    >&2 echo "VALUE is not set: trying ${NAME}"

    # Try to se the VALUE by getting the value of the variable named  i.e. SOME_DYNAMYCALY_NAMED_VARIABLE_1.
    # The magic is in the ${!NAME}, which sets the value of $SOME_DYNAMYCALY_NAMED_VARIABLE_1 to the VALUE variable.
    export VALUE=${!NAME}
done
{% endhighlight %}

The whole magic (and something new for me) is this:

{% highlight bash %}
${!NAME}
{% endhighlight %}
