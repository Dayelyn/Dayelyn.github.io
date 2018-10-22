---
layout: post
title:  "Tricks When Reading Json in Python"
date:   2018-10-22 16:55:51 +0800
categories: Tricks
tags: json python
description: Mainly demonstrate how to read and write json with the right format in Python. The tricks can be achieved from Google but I summarized a little bit here.
---

<script type="text/javascript" async src="//cdn.bootcss.com/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# Foreword

I have been on my work for a while, like about two months. When I was in university I mainly wrote code with Python, but now I have to get familiar with C++ as soon as possible - almost all backend engineering codes are written in C++.

However, Python is a great tool for fast development. If I would like to write a simulator or information generator, Python is still my first choice. Last week when I wrote a position information generator for containers, I have met several traps and finally overcome them. Here is a brief summary.

# Problem Description

* When saving data in the style of json, it can not be deserialized in the right format in C++;
* When appending a dict object in a list in python, all elements in the list will be changed at the same time.

# Solution

For the first problem, users can specify the json style with parameters in Python. The details can be checked [here](http://www.runoob.com/python/python-json.html).

For the second problem, it is because I directly push the object into the list. Actually it is a pointer to the object. The details can be checked [here](https://blog.csdn.net/sinat_21302587/article/details/72356431). Just use ```copy.deepcopy()``` to solve the problem.

# Summary

This is a short article since I didn't have enough time to record a simple proble in detail. But wrting down is always better than memory, as a Chinese old saying presents.