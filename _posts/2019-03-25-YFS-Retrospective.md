---
title: YFS - A Distributed Filesystem Retrospective - WIP
date: "2019-03-25T17:44:03"
excerpt: ""
layout: post
tags: [retro, gif, frequency, visualizer]
project: true
---
As a nearly semester-long project for a distributed systems class, we were to
implement a distributed filesystem called YFS, based on an [MIT 6.824 project](https://pdos.csail.mit.edu/archive/6.824-2012/labs/)
Below I'll outline the good, bad, and ugly about the project, as well as some 
lessons I learned.

![YFS-Diagram](./yfs.jpg)

### The Lay of the Land
The first big mistake I made during this project involved my outlook on C++.

Now, having at this point completed two C based courses, one on low level computing,
with an introduction to C, and the next Operating Systems, where we rebuilt 
some system fundamentals ourselves in C, I felt somewhat confident with things
like pointers and raw memory management. And, here comes the mistake, I took that
confidence and extended it into C++, thinking something along the lines of "C++ is just
C with classes". 

This statement is in a sense "technically" true, but also very misleading. The reality of writing 
C++, especially more modern C++11, or 14 (or 17), is very different to writing C. 

This became especially apparent when we were discussing our solutions to the first portion 
of the project in class. Where some of my classmates had made quick work of adding 
the extent functionality required by using C++ STL utilities like maps and vectors, I had
done everything C-style with arrays and far more pointer headaches than necessary. 

To be continued...





