---
title: "Let's add some color into my terminal"
excerpt: "Try to replicate the behaviour of "
tags: 
  something
date: 2020-08-05 18:26:00 0000
categories: things
---
There are lots of "custom" tools that allows some colorization of the terminal, like:
- lolcat (link)
- lolcat the python version (link)

and many others, but I'll focus on these two as they contain the very simple behaviour I'm trying to emulate: *rainbow colors*!!!

# Understanding the behaviour
In order to understand what I'm trying to achieve, let's unpack what those tools do:
1. you can "pipe" the tool after your command, and it'll colorize it and print back the output into the terminal.
2. you can add some flags in order to define different styles of colors, like rainbow, flat color, gradient (maybe?)
3. you can select the file you want to colorize and print (downside is that you cannot use internal editors to see the colorized output, so if the file is quite big you'll see pretty much nothing interesting)
4. 