---
title: Weird Exception While Testing
layout: post
author: Žan Anderle
date:   2016-03-20 10:50:00 +0100
categories:
    - til
tags:
    - django
    - testing
---

Tracebacks are crucial part of a coding process. Even more so when writing tests. So when traceback gives you inaccurate information, it can really mess with you. Last week I spent way too long on an exception that should have been trivial. The reason? Incomplete traceback.

I wrote a test for email templates in `test_email_templates.py` with the following imports:

{% highlight python %}
# -*- coding: utf-8 -*-
from django.test import TestCase
from django.template.loader import get_template, TemplateDoesNotExist
from django.utils.translations import override
from django.conf import settings
from django.contrib.sites.models import Site
{% endhighlight %}

After I wrote the test, I ran the test which resulted in a weird exception

{% highlight bash %}
$ ./manage.py test my_app.tests.test_email_templates
Traceback (most recent call last):
  File "manage.py", line 10, in <module>
    execute_from_command_line(sys.argv)
  File "/Users/zanderle/.virtualenvs/env/lib/python3.4/site-packages/django/core/management/__init__.py", line 351, in execute_from_command_line
    utility.execute()
  File "/Users/zanderle/.virtualenvs/env/lib/python3.4/site-packages/django/core/management/__init__.py", line 343, in execute
    self.fetch_command(subcommand).run_from_argv(self.argv)
  File "/Users/zanderle/.virtualenvs/env/lib/python3.4/site-packages/django/core/management/commands/test.py", line 30, in run_from_argv
    super(Command, self).run_from_argv(argv)
  File "/Users/zanderle/.virtualenvs/env/lib/python3.4/site-packages/django/core/management/base.py", line 394, in run_from_argv
    self.execute(*args, **cmd_options)
  File "/Users/zanderle/.virtualenvs/env/lib/python3.4/site-packages/django/core/management/commands/test.py", line 74, in execute
    super(Command, self).execute(*args, **options)
  File "/Users/zanderle/.virtualenvs/env/lib/python3.4/site-packages/raven/contrib/django/management/__init__.py", line 41, in new_execute
    return original_func(self, *args, **kwargs)
  File "/Users/zanderle/.virtualenvs/env/lib/python3.4/site-packages/django/core/management/base.py", line 445, in execute
    output = self.handle(*args, **options)
  File "/Users/zanderle/.virtualenvs/env/lib/python3.4/site-packages/django/core/management/commands/test.py", line 90, in handle
    failures = test_runner.run_tests(test_labels)
  File "/Users/zanderle/.virtualenvs/env/lib/python3.4/site-packages/django/test/runner.py", line 209, in run_tests
    suite = self.build_suite(test_labels, extra_tests)
  File "/Users/zanderle/.virtualenvs/env/lib/python3.4/site-packages/django/test/runner.py", line 121, in build_suite
    tests = self.test_loader.loadTestsFromName(label)
  File "/usr/local/Cellar/python3/3.4.2_1/Frameworks/Python.framework/Versions/3.4/lib/python3.4/unittest/loader.py", line 114, in loadTestsFromName
    parent, obj = obj, getattr(obj, part)
AttributeError: 'module' object has no attribute 'test_email_templates'
{% endhighlight %}

This got me scratching my head and I started looking for possible reasons. I got nowhere for 10 minutes. Eventually I noticed something by chance.

{% highlight python %}
from django.utils.translations import override
{% endhighlight %}

There is no `translations` in `django.utils`. There is however, `django.utils.translation`. So simply changing it to

{% highlight python %}
from django.utils.translation import override
{% endhighlight %}

Got everything working nicely. I later realized, that I could have figured this out by either running all the tests (or all the tests in the app), or trying to import this specific test. Both of which returns the accurate traceback.

{% highlight python %}
In [1]: import my_app.tests.test_email_templates
---------------------------------------------------------------------------
ImportError                               Traceback (most recent call last)
<ipython-input-1-f8f5ee3652d4> in <module>()
----> 1 import my_app.tests.test_email_templates

/Users/zanderle/Projects/dmt/datafy/my_app/tests/test_email_templates.py in <module>()
      2 from django.test import TestCase
      3 from django.template.loader import get_template, TemplateDoesNotExist
----> 4 from django.utils.translations import override
      5 from django.conf import settings
      6 from django.contrib.sites.models import Site

ImportError: No module named 'django.utils.translations'
{% endhighlight %}

When running all the tests in an app:
{% highlight bash %}
======================================================================
ERROR: my_app.tests.test_email_templates (unittest.loader.ModuleImportFailure)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/usr/local/Cellar/python3/3.4.2_1/Frameworks/Python.framework/Versions/3.4/lib/python3.4/unittest/case.py", line 58, in testPartExecutor
    yield
  File "/usr/local/Cellar/python3/3.4.2_1/Frameworks/Python.framework/Versions/3.4/lib/python3.4/unittest/case.py", line 577, in run
    testMethod()
  File "/usr/local/Cellar/python3/3.4.2_1/Frameworks/Python.framework/Versions/3.4/lib/python3.4/unittest/loader.py", line 32, in testFailure
    raise exception
ImportError: Failed to import test module: my_app.tests.test_email_templates
Traceback (most recent call last):
  File "/usr/local/Cellar/python3/3.4.2_1/Frameworks/Python.framework/Versions/3.4/lib/python3.4/unittest/loader.py", line 312, in _find_tests
    module = self._get_module_from_name(name)
  File "/usr/local/Cellar/python3/3.4.2_1/Frameworks/Python.framework/Versions/3.4/lib/python3.4/unittest/loader.py", line 290, in _get_module_from_name
    __import__(name)
  File "/Users/zanderle/Projects/dmt/datafy/my_app/tests/test_email_templates.py", line 4, in <module>
    from django.utils.translations import override
ImportError: No module named 'django.utils.translations'
{% endhighlight %}

I still haven't figured out why this doesn't show up when running a specific test. Useful to know though.