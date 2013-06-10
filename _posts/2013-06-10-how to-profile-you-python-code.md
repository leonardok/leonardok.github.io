---
layout: post
title: How to profile you python code
tags: [profiling, python, qcachegrind, cProfile]
description: Profile your python code to see where are the bottlenecks
last_updated: 06/10/2013
---

This is a brief explanation of how you can profile your code, in order to improve your benchmarks.

### Requirements

    pip install pyprof2calltree
    
    
### Who is taking my cheese?

There are lots of stuff that will try to eat your CPU time. You are programming these things, and you are using
things others did, all trying to eat out your CPU time. You need to find out where they are, if you want to sleep
at night. For instance, I know you are not using ``range``, but look what happens if you want to use it instead
of the new ``xrange`` (for clarification, you probably should not be doing ``range`` or ``xrange`` at all):

    time python -m cProfile matriz.py
             5003 function calls in 2.570 seconds
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    2.417    2.417    2.570    2.570 matriz.py:3(<module>)
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
         5001    0.153    0.000    0.153    0.000 {range}
    
    real    0m2.605s
    user    0m2.592s
    sys     0m0.000s

    #################################

    time python -m cProfile matriz.py
             2 function calls in 2.381 seconds
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    2.381    2.381    2.381    2.381 matriz.py:3(<module>)
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
    
    real    0m2.412s
    user    0m2.384s
    sys     0m0.016s
    
    
    1 #!/bin/python
    2
    3 z = 0
    4 for j in xrange(5000):
    5     for i in xrange(5000):
    6         z += 1
