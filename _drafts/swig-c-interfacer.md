---
title: "SWIG - interface for C and C++"
categories: programming 
tags: C C++ 
---

## SWIG
> [SWIG] is a software development tool that connects programs written in C and C++ with a variety of high-level programming languages. SWIG is used with different types of target languages including common scripting languages such as Javascript, Perl, PHP, Python, Tcl and Ruby. 

~~~~
% swig -go example.i
% gcc -fpic -c example.c example_wrap.c -I /usr/local/include
% gcc -shared example.o example_wrap.o -o example.so

~~~~

[SWIG]: https://swig.org/index.html
[SWIG Master Class]: http://www.dabeaz.com/SwigMaster/index.html
