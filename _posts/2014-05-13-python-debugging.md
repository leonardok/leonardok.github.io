---
layout: post
title: Python Debugging Notes
tags: [python, debug, debugging, celery]
description: Python Debugging Notes
last_updated: 05/13/2014
---

The most basic way for debugging python would be to print variable information
during the code execution, but we all know that this has some implications that
make the rest of development a more sadistic process, as it introduces code
that will have to be removed before the code goes to products.


Debugging live code can be improved by using breakpoints, calling the debugger
within the code brings up a shell where you can use the debugger commands. Some
of the tools I use are (you can install them with pip, package names are in parenthesis):

* Python Debugger (pdb) - [https://docs.python.org/2/library/pdb.html](https://docs.python.org/2/library/pdb.html) (for python 2.x)
* IPython Debugger (ipdb) - [https://pypi.python.org/pypi/ipdb](https://pypi.python.org/pypi/ipdb)
* PuDB (pudb) - [https://pypi.python.org/pypi/pudb](https://pypi.python.org/pypi/pudb)



The pdb and ipdb work are fairly similar, but the ipdb has some great features
that come with ipython, such as autocomplete and highlights.

The most different here is the PuDB, that features some kind of a full UI for
debugging, see the image bellow. 

![](/images/debugging_pudb_screenshot.png)

#### test.py

Now I wrote the following code the I need to step-by-step debug, so let's use
each tool to check the values for `i`.

{% highlight python linenos %}
#!/bin/python
    
def my_loop():
    for i in range(20):
        import pdb; pdb.set_trace()
        print i
    
def main():
    my_loop()
    
if __name__ == "__main__":
    main()
{% endhighlight %}


### pdb

To create a breakpoint using pdb just past the following code:

	import pdb; pdb.set_trace()

That puts us into the shell:

	python test.py
	> .../tmp/test.py(6)my_loop()
	-> print i
	(Pdb) _

Now if we enter `?` and hit enter we can see the help for the documented commands. 

	(Pdb) ?
	
	Documented commands (type help <topic>):
	========================================
	EOF    bt         cont      enable  jump  pp       run      unt
	a      c          continue  exit    l     q        s        until
	alias  cl         d         h       list  quit     step     up
	args   clear      debug     help    n     r        tbreak   w
	b      commands   disable   ignore  next  restart  u        whatis
	break  condition  down      j       p     return   unalias  where
	
	Miscellaneous help topics:
	==========================
	exec  pdb
	
	Undocumented commands:
	======================
	retval  rv

One command that can save you when you are lost is the `list` command. It
basically prints the source code, for your current location, with a few lines
before and after.

	(Pdb) list
	  1     #!/bin/python
	  2
	  3     def my_loop():
	  4         for i in range(20):
	  5             import pdb; pdb.set_trace()
	  6  ->         print i
	  7
	  8     def main():
	  9         my_loop()
	 10
	 11     if __name__ == "__main__":

Basic usage commands:

* `c` or `continue` to go to the next break point (if there is no other break point it will finish execution);
* `n` or `next` will go to next line
* `s` or `step` will make a "step into" kind of thing
* `u` or `up` will "step up"

You can also call your script as argument for pdb, it will start it in the
debugger where you can use all commands from the beginning of your script.


### ipdb

`ipdb` is very similar to `pdb` but has all the cool features from the
`ipython` shell. From the start you will see, at least, better colors. It also
has the same commands as the `pdb`, so this is just a matter of test, or
project limitations. You can always enter `help` in the command line to get a
more detailed list of the "documented" commands.


### PuDB

Let's start our script from command line with `pudb` by calling `pudb test.py`.


![](/images/debugging_pudb_screenshot_2.png)


You can see that it opens a really nice and busy view of the code we are in.
You can call help by typing "?". The hotkeys n, s, c and u have the same
meaning, so it feels like home. 

One cool thing you can do with this that is easier in PuDB is that by hitting
`b` you can set a break point where the cursor is. You can also call `<ctrl>+p`
to edit your preferences. Even set the default python shell to ipython, if you
want.

To call it, as a breakpoint, from within your code, you can paste the line
bellow where you want it to pause execution and bring up the interface.

	from pudb import set_trace; set_trace()
	


### Celery Remote Debugging

For the lack of a better place to add this note, and as this is also debugging.
The documentation is at
http://celery.readthedocs.org/en/latest/tutorials/debugging.html.

To remote debug celery tasks you basically open give rdb a port, then you can
connect with telnet.

	from celery.contrib import rdb
	rdb = rdb.Rdb(port=6899, port_search_limit=100, port_skew=0)
	
	rdb.set_trace() # set breakpoint


