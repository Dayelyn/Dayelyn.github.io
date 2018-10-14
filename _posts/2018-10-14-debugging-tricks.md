---
layout: post
title:  "Debugging Tricks of C++"
date:   2018-10-14 20:51:51 +0800
categories: Tricks
tags: C++ Debug
description: Traps I met when set/get key-values when using MySQL and Redis. The tricks can be achieved from Google but I summarized a little bit here.
---

<script type="text/javascript" async src="//cdn.bootcss.com/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# Foreword

That is my second update on Sunday. Actually today is an extra working day in purpose of plan review - although the review can still move on without me. But it is good for me to summarize my daily work last week since I am still in learning process. 

The topic of this article is the trick of dealing with a famous error message 
```
Segmentation fault (core dumped)
``` 
in C++. I believe most programmers have suffered from this error message in variety of ways, like invalid reference or null pointer. After today's catastrophy I finally have decided to fight against it and eventually have a pretty good solution to it.

# Problem Description

My situation today is that when I tried to run a compiled testing environment **task_scheduler**, the output shows
```
sow 0: 5 3 6 
sow 1: 2 7 8 4 
task id: 2 from node: 14 to node: 11
task id: 3 from node: 18 to node: 1
task id: 4 from node: 9 to node: 19
task id: 5 from node: 0 to node: 8
task id: 6 from node: 7 to node: 10
task id: 7 from node: 8 to node: 7
task id: 8 from node: 18 to node: 3
straddle inital position: 1
straddle inital position: 2
straddle inital position: 5
straddle inital position: 5
one local search loop!
child: | 2  | 5  | 7  | 3  | 6  | 8  | 4  | 3  | 0  | 0  | 4  |  
child: | 7  | 6  | 2  | 8  | 4  | 3  | 5  | 0  | 2  | 4  | 1  |  
child: | 2  | 8  | 7  | 4  | 3  | 6  | 5  | 2  | 3  | 1  | 1  |  
child: | 8  | 4  | 5  | 3  | 2  | 6  | 7  | 0  | 2  | 3  | 2  |  
child: | 5  | 3  | 8  | 2  | 6  | 4  | 7  | 3  | 1  | 2  | 1  |  
child: | 5  | 7  | 4  | 2  | 8  | 6  | 3  | 3  | 1  | 2  | 1  |  
child: | 3  | 2  | 6  | 5  | 8  | 4  | 7  | 3  | 1  | 2  | 1  |  
child: | 5  | 3  | 2  | 6  | 7  | 8  | 4  | 2  | 4  | 1  | 0  |  
child: | 7  | 3  | 6  | 5  | 2  | 8  | 4  | 3  | 1  | 1  | 2  |  
child: | 5  | 6  | 3  | 2  | 7  | 8  | 4  | 2  | 0  | 1  | 4  |  
Segmentation fault (core dumped)
```
I have created a population of 10 children for the genetic algorithms with **GAlib** and they were initialized correctly due to the output. After the setup of the GA algorithm, **GAlib** needs to call a method `ga.initialize()` to initialze the population. The output sentence of 10 children representation was written in the inherented initialzation function. As a result, you can see that after the initialization the program is down.

The most tricky point to debug this problem is that the flag is no longer effective. Even you setup breakpoint in the initialzation, you can not locate where the problem happened.

# Solution
I totally have no idea about how to debug segmentation fault, since I am a newbie in C++. When I search it on Google, surprisingly I find people have already dig it deeply and I can have a great view on how to deal with it.

The core tools are **core** file and **gdb** tools. I will describe them separately.

## Core File

The error **core dumped** actually mean that the system saves the process information on the disk when the process is terminated with errors. Hence, to analyse the **core file** is important for solving the problem.
```
$ ulimit -a
core file size          (blocks, -c) 1024
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 62684
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 62684
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```
On linux you can use `ulimit -a` to check if you have a core file. Generally the core file size would be 0. To make it available, you need to run `ulimit -c 1024` to specify a size for the core file(here 1024 can be any value).

Now run your 'faliure' program again and you will find a **core** file in the directory of your compiled program. To analyse it, use the command `gdb core-file core`
```
$ gdb core-file core
GNU gdb (Ubuntu 7.11.1-0ubuntu1~16.5) 7.11.1
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
core-file: No such file or directory.
[New LWP 22480]
Core was generated by `./optimized_task_schedule_platform'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0  0x00007f60596f4894 in ?? ()
```

Instead of a lonely 'segmentation fault', you can read more information in core file. In my case it points out that the program is terminated with **signal SIGSEGV**. What is it?

## Signal SIGSEGV

Generally a signal SIGSEGV means the program has an invalid memory reference. We can use a command
```
$ man 7 signal | grep SEGV
       SIGSEGV  and  SIGFPE,  generated as a consequence of executing a specific machine-
       SIGSEGV      11       Core    Invalid memory reference
```
to check and the output shows that we definitely have an invalid memory reference.

## GDB Tool

Up to now we seem to be stuck in an invalid memory reference. For a small program you can find the invalid reference 'by eyes'. But for a large scale project, it is very difficult to debug sentence by sentence. Therefore, we have to introduce **gdb** tool to debug.

**gdb** is a powerful tool for debugging. The introduce and brief tutorial can be found in [reference1](https://www.cnblogs.com/TianFang/archive/2013/01/20/2868889.html) and [reference2](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/gdb.html).

In my case I run the compiled program with gdb
```
$ gdb ./task_scheduler
(gbd) l         // code list
(gdb) b (line)  // set break point in (line)
(gdb) c/u/s     // c for continue, u for until, s for step
```

use the combination of commands mentioned above, finally I locate the problem
```
TaskGenome::Evaluate (g=...) at src/GeLib/TaskGenome.cpp:676
676	    float total_cost = solver.Evaluation(g, straddle_data, task_data);
(gdb) s

...

std::vector<TaskData, std::allocator<TaskData> >::vector (this=0x7fffffffcc60, 
    __x=std::vector of length 7, capacity 7 = {...})
    at /usr/include/c++/5/bits/stl_vector.h:321
321	      { this->_M_impl._M_finish =
(gdb) u
325	      }
(gdb) u

Program received signal SIGSEGV, Segmentation fault.
0x000000000043769e in std::vector<double, std::allocator<double> >::size (this=0x0)
    at /usr/include/c++/5/bits/stl_vector.h:655
655	      { return size_type(this->_M_impl._M_finish - this->_M_impl._M_start); }
(gdb) quit

```

Now it shows clearly that `TaskGenome.cpp` calls `Evaluation` while in `Evaluation` a variable with data structure `std::vector<double, std::allocator<double>>`, which is actually `vector<vector<double>>` has not initialzed when it is called (I got this conclusion by checking the code).

# Summary
All tricks mentioned above are helpful for locating the error in a gradually smaller scale. Be patient and focused on the details, problems can always be solved eventually.