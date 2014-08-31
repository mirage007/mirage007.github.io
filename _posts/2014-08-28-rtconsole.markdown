---
layout: post
title:  "rtconsole -- Runtime Console"
date:   2014-08-28 23:15:13
categories: runtime console python ipython
---
{% include JB/setup %}

Have you ever wanted to see what your python application was doing while it was running? What about inspecting the state of any of its variables at runtime, and perhaps changing them in real time? And why not do it from a really nice interface like an IPython console while you are at it?

With the advent of cloud computing, more and more applications are adopting a SaaS framework. Loosely coupled applications talk to each other in real time, and it becomes increasingly difficult in such a world to easily "isolate all the inputs" and to make reproducible test cases without investing a large amount of time to log all intermediate states of your application. The inputs may come from other web services which could frankly be ANYTHING. 

This is further complicated in the domain of data analysis where it's not so much that it's throwing out an error as much as the results are just a little suspect. In that context, it might be much more intuitive to get immediate feedback by checking/changing some inputs and see how it behaves than to create a full blown test case for each incremental change.

Even if you have full control of the application, a typical workflow could be something like this:

1. stopping the application
2. writing logging statements for something suspect
3. rerunning the application and wait for something weird to happen
4. piece together the information from the log, and building a precise test case
5. realizing that the issue isn't there, and  you need to log some other part of the application
6. repeat

Instead it could be more like:

1. start the application with rtconsole enabled
2. inspect the state and change some variables to see how the application responds
3. once you have a good idea of where the problem is, just write some commands to dump the states you care about right there and then

That's the driving motivation behind rtconsole. 

Getting Started
---------------

Copy [rtconsole.py][rtconsole] into your project and run the `test_console.py` script, simplified version reproduced below:

    import time
    from rtconsole import start_console
    start_console(locals())
    time.sleep(5) # takes some time for it to start
    t = 0
    hello = lambda x: x**2
    while True:
        t += 1
        s = hello(t)
        time.sleep(1)
    print t # this will print to the ipython console

Assuming no other ipython sessions are running, you can now access the script via the IPython console:

    :~/Sites/env/default/project/rtconsole mirage007$ipython console --existing

    Python 2.7.5 (default, Sep  7 2013, 18:00:40) 
    Type "copyright", "credits" or "license" for more information.

    IPython 2.2.0 -- An enhanced Interactive Python.
    ?         -> Introduction and overview of IPython's features.
    %quickref -> Quick reference.
    help      -> Python's own help system.
    object?   -> Details about 'object', use 'object??' for extra details.

    In [1]: t
    ---------------------------------------------------------------------------
    NameError                                 Traceback (most recent call last)
    <ipython-input-1-b7269fa25085> in <module>()
    ----> 1 t

    NameError: name 't' is not defined

    In [2]: t
    hello
    Out[2]: 2

    In [3]: t
    Out[3]: 7

We can see that `t` wasn't initialized when we first asked for it, but eventually gets created and incremented

    In [4]: s
    Out[4]: 64

    In [5]: (t,s)
    Out[5]: (13, 169)

    In [6]: hello
    Out[6]: <function __main__.<lambda>>

    In [7]: hello?
    Type:        function
    String form: <function <lambda> at 0x1117b87d0>
    File:        /Users/mirage007/Sites/env/default/project/rtconsole/test_console.py
    Definition:  hello(x)
    Docstring:   <no docstring>

    In [8]: hello??
    Type:        function
    String form: <function <lambda> at 0x1117b87d0>
    File:        /Users/mirage007/Sites/env/default/project/rtconsole/test_console.py
    Definition:  hello(x)
    Source:          hello = lambda x: x**2

Docstrings and code inspection works

    In [9]: hello = lambda x: x+5

    In [10]: hello??
    Type:        function
    String form: <function <lambda> at 0x1105cc410>
    File:        /Users/mirage007/Sites/env/default/project/rtconsole/<ipython-input-9-aa74116e9749>
    Definition:  hello(x)
    Source:      hello = lambda x: x+5

It even works when the function has been defined on the fly.

    In [11]: (t,s)
    Out[11]: (41, 46)


And `s` is now computed using the new hello function

To close out of the Kernel, press `Ctrl-\` 

Some key points:

- The print statement was redirected to the IPython console. 
- We can also change it as it runs, and the app just keeps running with the new code
- We still have all the benefit from using IPython for introspection. 

How this works
----------

rtconsole patches the IPython kernel so that it can be run in a python thread. The IPython zmq kernel cannot do that as of IPython 2.2 because it uses the python `signal` module, which according to documentation, [can only be called on the main thread](https://docs.python.org/2/library/signal.html#module-signal). Not being aware of this while also not being so familiar with the internals of IPython, I found out the hard way while trying to hook an rtconsole and an IPython client together. *The failure of a `signal` call would crash the application in such a way that no exception was thrown.*

Limitations/Next Steps
----------

- `print` statements will get redirected to IPython once the console starts. But only flushes to the console once you type a command. There are also some performance considerations of using prints since they now have to be sent over the network, but in the context of debugging it may be acceptable.
- I also haven't found a way to redefine functions when the method uses *predefined* variables i.e. `hello = lambda x: t+5` would crash `test_console.py` with the error `"global name 't' is not defined"`
- When the application `test_console.py` terminates, the history manager throws an error, i haven't found this to impact anything but definitely would like to see if anyone has insights to address these issues.

With the domain of data analysis and cloud computing becoming increasingly available and easy to use, the lines between developer and user is becoming increasingly blurred. This tool aims to speedup the debugging process and make python applications less of a black box. Please let me know if this post was interesting or if you have ideas on how to make this tool more useful.

[rtconsole]: https://github.com/mirage007/rtconsole
