---
layout: post
title: Adding git branch to your PS1
tags: [bash, PS1, post, linux]
description: Add current git branch to your PS1 linux prompt
last_updated: 03/06/2013
---

I like command line; I use git; I always have exactly no idea in which branch 
I am.

### Adding the git branch to PS1 in bash:

Add this function to your .bashrc:
	function parse_git_branch {
		git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\1)/'
	}

It is divided into the execution of two scripts. The first ``` -e '/^[^*]/d' ```
will delete every line without a star. The second ``` -e 's/* \(.*\)/(\1)/' ```
will get the branch name replacing the star and space by nothing.

And that will return our branch name. Cool. Next: embbed it into PS1.

PS1 is a very cool feature of bash. And I call it feature because it
really helps me. Anyway, you should put something like that:

	PS1='[\u@\h \w]$(parse_git_branch)\$ '

Well this is a bit of my PS1, in my real one, there colors everywhere. You 
can check it out on [this link to my github account](https://github.com/leonardok/dotfiles-bash)
Just to let you know what means what, if you are reading this and have no 
ideia nor can find the man page :-)

+ ```\u``` - your username
+ ```@ ```  - that is an @ sign, really
+ ```\h``` - the host name
+ ```\w``` - the relative path you are in


For a full reference to all PS1 options check out 
[this link for the bash man page](http://linux.die.net/man/1/bash)
or simply ``` $ man bash ``` in any linux system and look for "PROMPTING".
