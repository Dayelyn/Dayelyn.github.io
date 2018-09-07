---
layout: post
title:  "A* with the heuristic function of FastMap for path planning"
date:   2018-09-05 13:51:51 +0800
categories: Literature_Study
tags: Path_Planning Genetic_Algorithm MTSP
description: The third literature study.
---

<script type="text/javascript" async src="//cdn.bootcss.com/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# Artical Title

A new crossover approach for solving the multiple travelling salesmen problem using genetic algorithms. By Shuai Yuan etc.

# Foreword
I have been suffering a heavy cold from yesterday. I guess it is most likely because I prefer standing under the air conditioner right after I step into my office. The summer in Shanghai is too warm compared with Stockholm, but much more interesting :)

# Abstract
To solve the multiple travelling salesman problem(MTSP), a genetic algorithm(GA) with two-part chromsome representation is considered as an effective way. However, in the original paper the author uses the crossover method of the ordered crossover operator(ORX) for the first part, and the asexual crossover operator(A) for the second part and they are proved to be less effective than the method TCX mentioned in this paper.

We are going to start with the MTSP. Then GA is introduced with several crossover operator and the last is the TCX.

# MTSP

I think in [this article](https://neos-guide.org/content/multiple-traveling-salesman-problem-mtsp) the problem is formulated clearly.

# Genetic Algorithm

Countless tutorial can be found on the Internet for the introduction of GA. [Here](https://www.tutorialspoint.com/genetic_algorithms/) is a great tutorial in my opinion to give a overview of the genetic algorithm.

Also for Chinese friends, a youtube list from [Morvan Zhou](https://www.youtube.com/playlist?list=PLXO45tsB95cJyeE6BgkApUbAREpkoPDvG) is a great tutorial with both theory and practical code.

# A New Crossover Approach

## Step 1: Representation

As can be seen in the following figure, the first part of length $n$ in the chromsome represents $n$ cities and the second part of length $m$ represents $m$ salesmen. 

Labels in the first part represent the label of cities, and these in the second part represent cities per salesman visits. For example in the figure, in the second part, the pair $(4,3,2)$ means salesman 1 needs to visit 4 cities, accordingly $6,9,1,7$ in the first part. Then same representations for other two salesmen.

![Imgur](https://i.imgur.com/MasUqvA.png)

Then we need both Mon and Dad and you can choose either as the base of offsprings.

![Imgur](https://i.imgur.com/px4mp2A.png)

## Step 2: Gene Segments Selection in the first Part from Base

In the paper we only focus on the crossover step in the GA. When we step into the crossover step, the first thing we should do is to selection which genes we are going to crossover. As the author says, we need to randomly choose gene segments for each salesman. For example, in the following figure, if we choose Mon as the base, $(7,5)$ is selected for salesman 1, $(8,4)$ is selected for salesman 2 and $(1)$ is selected for salesman 3. Then the number of selected genes for them is recorded as $(2,2,1)$.

![Imgur](https://i.imgur.com/t7zdiuV.png)

## Step 3: Shuffle the Remaining Genes According to Another Parent

From the above figure we can see that $(9,6,2,3)$ are the remaining genes. Acoording to the genes from Dad in the following figure, we can shuffle $(9,6,2,3)$ to $(2,3,9,6)$.

![Imgur](https://i.imgur.com/TVqKzc1.png)

## Step 4: Randomly Assign the Remaining Genes to Each Salesman

Recall in step 2 we assign 2 cities for salesman 2, 2 to salesman 2 and 1 for salesman 3. Now only $2+2+1=5$ cities are assigned, $(2,3,9,6)$ still needs to be assigned to salesmen. Here we can randomly generate numbers of assignment like $(4,0,0)$ (means 6 cities for salesman 1, 2 for salesmen 2 and 1 for salesman 3) and so on. In the paper the author chooses $(2,1,1)$ so that the result can be retreived from the following figure:

![Imgur](https://i.imgur.com/ESgJ8O9.png)

# Step 5: Organization

Recall in step 1 of how the representation is. Now the first part of the child chromsome has been constructed. For the second part, we know that in step 2 we select $(2,2,1)$ and in step 4 we assign $(2,1,1)$ for each salesman. Therefore, the second part can be constructed in the following figure:

![Imgur](https://i.imgur.com/ikD2X5F.png)

# Algorithm and Implementation

They can be added later.

## Application Scenario

In the introcuction the author points out that the MTSP is actually a general model for lots of problems. The Viehcle Scheduling Problem is one of them. With respect to the business of our company, I guess the algorithm is used to plan an available task for each straddle so that they can fulfil the task requirements.

## Something Confused

I think it is very clear in both theory and algorithm. The important thing for me is to have an available implementation of the algorithm.