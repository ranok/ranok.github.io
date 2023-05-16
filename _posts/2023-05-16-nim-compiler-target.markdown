---
layout: post
title:  "Nim compiler target specific code"
date:   2023-05-16 08:55:04
categories: nim
---
As I've started developing more code in [Nim](https://nim-lang.org/), I've found myself spending a lot of time in the Gitter chat, on old forum posts, and using too much trial and error.
For everything I spend too much time on, I want to put my findings out there to hopefully help others. Today's topic is very simple, but took a while to figure out: including/excluding
code based on the compilation target. I really like Nim because of its ability to output both C/C++ or JS code, having a single codebase is useful when you want to reuse core
routines. Nowhere did I find this documented, but based on the compilation target, the compiler defines either `c` or `js`. This comes in handy when using libraries that only target
one output type, such as `re`.

Now you can have code like this:
```
when defined(c):
     import re
when defined(js):
     import jsre
````

On my todo list is to specifically make a generic RE library that has the same API and works for both C and JS targets, but at least there's now a simple way to differentiate code between
targets.
