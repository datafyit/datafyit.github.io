---
layout: post
author: Uroš Trebec
title:  "TIL: docker-compose creates a new container on every run"
date:   2016-03-17 10:50:00 +0100
categories:
    - til
tags:
    - docker
    - docker-compose
---

We use docker-compose extensively while developing Datafy.it services.

We all know running `docker-compose up` creates a new container `project_service_1`. This container is reused when the command is rerun.

Today I learned that each `docker-compose run` creates new container too. It's created with the name `project_service_run_X`, X being the sequence number.

Sooner or later, the container list can looks like this:

{% highlight bash %}
$ docker ps -a --filter "name=run" --filter "status=exited"

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
1d8f400747f9        excelsior_django    "/entrypoint.sh pytho"   20 hours ago        Exited (0) 20 hours ago                       excelsior_django_run_6
2ecc0fc809ae        excelsior_django    "/entrypoint.sh pytho"   20 hours ago        Exited (1) 20 hours ago                       excelsior_django_run_5
2109c22189cc        db4bad08bc33        "/entrypoint.sh pytho"   20 hours ago        Exited (1) 20 hours ago                       excelsior_django_run_4
ddee814711e4        ff97abc48153        "/entrypoint.sh pytho"   20 hours ago        Exited (1) 20 hours ago                       excelsior_django_run_3
cf51961d0b15        ff97abc48153        "/entrypoint.sh pytho"   20 hours ago        Exited (0) 20 hours ago                       excelsior_django_run_2
590987e41124        ff97abc48153        "/entrypoint.sh pytho"   20 hours ago        Exited (0) 20 hours ago                       excelsior_django_run_1
{% endhighlight %}

Fortunately, this mess can be cleaned:

{% highlight bash %}
$ docker ps -a -q --filter "name=run" --filter "status=exited" | xargs docker rm
{% endhighlight %}

Or avoided in the first place, by using the `--rm` option:

{% highlight bash %}
$ docker-compose run --rm service /bin/bash
{% endhighlight %}
