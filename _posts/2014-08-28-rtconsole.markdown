---
layout: post
title:  "rtconsole -- Runtime Console"
date:   2014-08-27 23:15:13
categories: runtime console
---
{% include JB/setup %}

Have you ever wanted to see what your application was doing while it was running? What about inspecting the state of any of its variables at runtime, and perhaps changing them in real time? And why not do it from a really nice interface like an IPython console while you are at it?

That's the driving motivation behind rtconsole. So how do you get started? Copy [rtconsole.py][rtconsole] into your project and use it by typing this:  
  
{% highlight python %}
##in your python script
from rtconsole import start_console
start_console(locals())
#some more computation here...
{% endhighlight %}

Alternatively, for a ready made example you can run the `test_console.py` script.

Assuming no other ipython sessions are running, you can now access the script via the IPython console using this command:
{% highlight bash %}
> ipython console --existing
{% endhighlight %}


How this works
==========

rtconsole overwrites the IPython kernel so that it can be run in a python Thread. The only thing that gets in the way of that is the redirection of system signals. I don't quite understand what those line do other than having found that it completely kills the kernel if those signals are generated from a non Main thread.

One other nifty feature or bug depending how you look at it is that any `print` statement will get redirected to IPython once the console starts, which makes for very difficult debugging while I was trying to make this example work.

Lastly, I haven't found out a way to close out of the IPython console without also killing the Kernel.

Please let me know of your thoughts
[rtconsole]: https://github.com/mirage007/rtconsole
