---
layout: post
title: "Surprise order sensitivity in Python"
lang: english
---

I'm in a research project that uses Python as the primary language for
implementations and I've run into a very weird problem twice now. The issue is
that some modules turn your code order sensitive for no apparent reason. 

Here's some snippets from a script that declares Celery tasks. TO ME, identical
from a behavior standpoint. However, the first one doesn't print anything. Why?
I don't know.

```python
    @shared_task(bind=True)
    def start(self):
        print("Hello world")
        navegador = webdriver.Remote('http://firefox:4444/wd/hub',desired_capabilities=DesiredCapabilities.FIREFOX)
        # ...
```

```python
    @shared_task(bind=True)
    def start(self):
        navegador = webdriver.Remote('http://firefox:4444/wd/hub',desired_capabilities=DesiredCapabilities.FIREFOX)
        print("Hello world")
        # ...
```

I ran into this issue earlier this year using OpenCV's DNN module.
In this case, running a subprocess after forwarding the network led to
repeated results. I even posted a code to reproduce the problem in an
[issue on the opencv repo](https://github.com/opencv/opencv/issues/19643#issuecomment-875222766),
but at the time of the writing this nobody has posted a definitive
explanation or solution.

My guess is that this is related to how `subprocess` is implemented on Linux or
perhaps how those subprocesses get started. It could also be a weird interaction
between Python subprocesses and native extension threads (OpenCV makes use of
OMP, TBB and plain threading).

