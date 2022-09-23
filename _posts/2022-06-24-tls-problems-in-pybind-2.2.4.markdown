---
layout: post
title: Callbacks, threads and thead-local storage on pybind11 2.2.4
---

_Disclaimer: This was also posted in the [pybind discussion board](https://github.com/pybind/pybind11/discussions/4029)_

In pybind11 2.2.4, invoking a Python callback from a C++ function running in a Python-created thread will result in a situation where the Python callback will not have access to thread-local storage initialised on the same thread that invoked the C++ function:

```
$ pip install cppimport "pybind11==2.2.4" &>/dev/null && rm cpp_module*cpython*so && python tlstest.py 2>/dev/null
Current pybind11 version is 2.2.4
From module..........: tls.value = hello world
From thread (Python).: tls.value = hello world
From thread (C++)....: tls.value = <undefined>
```

The issue was probably resolved by [PR #1211](https://github.com/pybind/pybind11/pull/1211) (see [this test code from the PR](https://github.com/pybind/pybind11/pull/1211/files#diff-262299f8dc5f31a314eeee23329842134461b9524aa535a2860252dbf78c6033R20-R49)) which got merged into pybind 2.3.0:

```
$ pip install cppimport "pybind11==2.3.0" &>/dev/null && rm cpp_module*cpython*so && python tlstest.py 2>/dev/null 
Current pybind11 version is 2.3.0
From module..........: tls.value = hello world
From thread (Python).: tls.value = hello world
From thread (C++)....: tls.value = hello world
```

## Commands to reproduce

* For pybind 2.2.4: ` pip install cppimport "pybind11==2.2.4" &>/dev/null && rm cpp_module*cpython*so && python tlstest.py 2>/dev/null`
* For pybind 2.3.0: `pip install cppimport "pybind11==2.3.0" &>/dev/null && rm cpp_module*cpython*so && python tlstest.py 2>/dev/null `


## Code to reproduce:

Note 1: make sure you have python-dev (I'm using 3.9) and a compiler.  
Note 2: copy even the comments otherwise `cppimports` might not be able to correctly build the native module.

### `cpp_module.cpp`
```cpp
// cppimport
#include <pybind11/pybind11.h>
#include <functional>

namespace py = pybind11;

std::function<void()> g_cb;

PYBIND11_MODULE(cpp_module, m) {
    m.def("set_callback", [](py::object cb) {
        g_cb = [cb]() {
            py::gil_scoped_acquire _;
            cb();
        };
    });

    m.def("call_the_callback", []() { 
        py::gil_scoped_release _;
        g_cb();
    });
}

/*
<%
setup_pybind11(cfg)
%>
*/
```

### `tlstest.py`

```python
import functools
import sys
import threading

import pybind11

import cppimport.import_hook
import cpp_module

tls = threading.local()
tls.value = "hello world"

def run_in_thread(f):
    @functools.wraps(f)
    def thread_wrapper(*args, **kwargs):
        def wrapper(*args, **kwargs):
            tls.value = "hello world"
            f()

        t = threading.Thread(target=wrapper, args=args, kwargs=kwargs)
        t.start()
        t.join()

    return thread_wrapper


def print_tls_value():
    print(f"tls.value = {getattr(tls, 'value', '<undefined>')}")

@run_in_thread
def called_from_python():
    print_tls_value()

@run_in_thread
def called_from_cpp():
    cpp_module.call_the_callback()

print("Current pybind11 version is", pybind11.__version__)
cpp_module.set_callback(print_tls_value)

sys.stdout.write("From module..........: ")
print_tls_value()

sys.stdout.write("From thread (Python).: ")
called_from_python()

sys.stdout.write("From thread (C++)....: ")
called_from_cpp()
```

