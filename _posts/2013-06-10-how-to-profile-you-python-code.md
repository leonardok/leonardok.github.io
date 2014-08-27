---
layout: post
title: How to profile you python code
tags: [profiling, python, kcachegrind, cProfile]
description: Profile your python code to see where are the bottlenecks
last_updated: 08/27/2014
---

This is a brief explanation of how you can profile your python code.

### Requirements

    pip install pyprof2calltree line_profiler
    
    
### Who is taking my cheese?

There are lots of stuff that will try to eat your CPU time. You are programming these things, and you are using
things others did, all trying to eat out your CPU time. You need to find out where they are.


### Tools I use

The Python docs have a section about profilers, you should read it at 
[http://docs.python.org/2/library/profile.html](http://docs.python.org/2/library/profile.html).

[pyprof2calltree](https://pypi.python.org/pypi/pyprof2calltree/), which is a converter for cProfile 
to the kcachegrind graphical calltree analyser, to see nice and colourful schematics of our function calls.

[Kcachegrind](http://kcachegrind.sourceforge.net/html/Home.html), as I just said, is a tool for graphic 
analysis (but not only graphic) of your code calls. There is also a windows port for it.

[line-profiler](http://pythonhosted.org/line_profiler/) is the tool to use after you have some idea of
where the bottlenecks are. It can tell, line by line, where the most of the time is being spent. This is really
awesome and have helped me a lot.


### How to

DISCLAIMER: Remember this is just some pointers to what you can do to profile your code. You should
always refer to the official documentation for more details.

#### cProfile

Take the code you want to profile and run with:

    # Call cProfile
    python -m cProfile [-o output_file] [-s sort_order] myscript.py
    
    # Turn the profile into a callgraphtree
    pyprof2calltree [-i input_file] [-o output_file]


This is the very simple function we are testing.
    
    #!/bin/python
    
    z = 0
    for j in xrange(5000):
        for i in xrange(5000):
            z += 1


Profiling with cProfile:

    time python -m cProfile myscript.py
             2 function calls in 2.381 seconds
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    2.381    2.381    2.381    2.381 matriz.py:3(<module>)
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
    
    real    0m2.412s
    user    0m2.384s
    sys     0m0.016s
    
Running it without profiling gives us almost the same time.

    time python test.py
    
    real    0m2.363s
    user    0m2.340s
    sys     0m0.012s
    
    
#### kcachegrind

With kcachegrind you can see your call tree in a more human readable form, like the picture bellow.
The input file for kcachegrind is the output of the ``pyprof2calltree [-i input_file] [-o output_file]``
command.

OBS: There is also a windows port availiable at 
[http://sourceforge.net/projects/precompiledbin/](http://sourceforge.net/projects/precompiledbin/).

(Picture taken from the official website at 
[http://kcachegrind.sourceforge.net/html/Shot3.html](http://kcachegrind.sourceforge.net/html/Shot3.html))

![kcachegrind](http://kcachegrind.sourceforge.net/html/pics/KcgShot3.gif)
    

#### line_profiler and kernprof

line_profiler will add some time to construct the .lprof result, but anyway it adds some overhead to
the main execution. You will have to run ``kernprof`` and then ``python -m line_profiler``.

    $ time kernprof.py -l test.py
    Wrote profile results to test.py.lprof
    
    real    0m46.832s
    user    0m46.695s
    sys     0m0.028s

    #######################################

    $ python -m line_profiler test.py.lprof
    Timer unit: 1e-06 s

    File: test.py
    Function: test_method at line 3
    Total time: 23.4783 s
    
    Line #      Hits         Time  Per Hit   % Time  Line Contents
    ==============================================================
         3                                           # @profile
         4                                           def test_method():
         5         1            1      1.0      0.0      z = 0
         6      5001         2226      0.4      0.0      for j in xrange(5000):
         7  25005000     11056412      0.4     47.1          for i in xrange(5000):
         8  25000000     12419670      0.5     52.9              z += 1



### Memory Profiling

Easy to use profiler, that has the same decorators as the other profile tools. It is not a complete analyser but can give an overall picture of memory increments. [https://pypi.python.org/pypi/memory_profiler](https://pypi.python.org/pypi/memory_profiler)


## References

* [Python Line-by-line Profiler by Silas Sewell](http://silas.sewell.org/blog/2009/05/28/python-line-by-line-profiler-line_profiler-and-kernprof/)
* [http://docs.python.org/2/library/profile.html](http://docs.python.org/2/library/profile.html)
* [pyprof2calltree](https://pypi.python.org/pypi/pyprof2calltree/)
* [kcachegrind](http://kcachegrind.sourceforge.net/html/Home.html)
* [kcachegrind for windows](http://sourceforge.net/projects/precompiledbin/)
* [line-profiler](http://pythonhosted.org/line_profiler/)
