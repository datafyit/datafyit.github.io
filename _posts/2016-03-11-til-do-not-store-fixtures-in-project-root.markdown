---
layout: post
title:  "TIL Do Not Store Fixtures In Project Root"
date:   2016-03-11 13:29:37 +0100
categories: django tests
---

Always check for the same name files in your Django project.

If you have two fixtures with the same name, one in your app's fixtures folder and one in your project dir, it will use the one in your project dir when you run your app's tests.

So, if you run tests, expecting that your test suite will use `my_app/fixtures/companies.json`,
you're gonna have a bad time. It will actually use the one in your `root` directory.

{% highlight bash %} . ├── companies.json ├── my_app | ├── fixtures | └── companies.json {% endhighlight %}
