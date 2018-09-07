---
layout: post
title:  "Shortest Route Searching with Limited Conditions"
date:   2018-09-03 13:19:51 +0800
categories: Literature_Study
tags: Path_Planning
description: The first literature study. Introduce a method of searching the shortest path based on time consuming.
---

<script type="text/javascript" async src="//cdn.bootcss.com/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# Article Title

Shortest Route with Time Dependant Length of Edges and Limited Delay Possibilities in Nodes.
By j.Halpern

# Abstract

This is the the first paper of three I have been received from the company as the The literature study. **The main idea** of the paper is searching for the shortest route under specific conditions as following:

* Parking in a node of a directed graph is limited in a certain period;
* Not all desirable delay of departure time from a given node is permitted (almost the same with bullet 1);
* Travelling time along the edge leading from i to j depends on the departure time from i.

Lots of pernomena in the real life can be modelled as the problem with mentioned conditions. An example is a one-way road with traffic light system. 

# Problem Formulation

Define $G = (N,E)$ as an $n$-node finite directed graph, then we denote:

* $N = \\{1,...,n\\}$:            the set of all nodes;
* $E = \\{(i, j), i, j\in N\\}$:  the set of all directed edges;
* $N_k = \\{j \in N, (k,j) \in E\\}$: the set of all j that edge exists from $k$ to $j$;
* $N^*$:                        the set of known nodes in the shortest route from origin; 
* $N^{**}$:                     the set of all nodes that reachable from current node;
* $d_{ij}(t)$:                  travelling time from node $i$ to $j$;
* $P_i = \\{ [\alpha_i^h,  \beta_i^h], h = 1,...,H_i\\}$: the set of permitted parking time intervals at node i;
* $A_j$: the set of feasible arrival times to node $j$;
* $D_j$: the set of feasible departure times from node $j$, $D_j = g(A_j, P_j)$.


A feasible route from node $r$ to node $s$ in $G$ is featured by:

* ordered set of nodes $\\{r = i_1, i_2, ...,i_q = s\\}$ such that $(i_k, i_{k+1}) \in E$ for all $k = 1, ..., q-1$;
* ordered set of pairs of time $\\{(t_{i_1}^{'},  t_{i_1}^{"}),...,(t_{i_q}^{'},  t_{i_q}^{"})\\}$, where $t_j^{'}$ is the arrival time to node $j$ and $t_j^{"}$ is the departure time from $t_j$. 

Then if a node $i_k$ is feasible in the route, the requirements are:

* $(t_{i_k}^{'},  t_{i_k}^{"}) \in P_{i_k}$, indicating the delayed time in node $i_k$ should lies in the permitted time interval;
* $d_{i_ki_{k+1}}(t_{i_k}^{"}) < \infty$, indicating no discontinuity on the edge $(i_k, i_{k+1})$. If there is any, a new node can be construct to avoid the problem. **(The unvaild edge will be divided into two parts, which can be used to cut unavailable paths.)**

Then the problem can be formulated as: **finding a feasible route $\\{r = i_1, i_2, ...,i_q = s\\}$ from $r$ to $s$, minimizes $t_s^{'}.$**

# Solution

## Preliminary

A few conclusions should be specified before we introduce the algorithm to solve the problem:

* Since the parking and delay is limited in the problem, a cycle (visiting a node more than once) can be involved in the final route to find the shortest route;
* Due to bullet 1, it is necessary to update $A_j$ whenever $N^*$ is expanded;
* $D_j$ is constructed with $A_j$ and $P_j$. If $A_j \cap [\alpha_j^h,  \beta_j^h] = \varnothing$, indicating no parking at node $j$, then $D_j = A_j$; if $A_j \cap [\alpha_j^h,  \beta_j^h] \not = \varnothing, Min \\\{ A_j \cap [\alpha_j^h,  \beta_j^h] \\\} = t^{'}$, indicating parking occurs at the node $j$ and $t^{'}$ is the smallest time point, then $\forall t: t^{'} \leq t \leq \beta_j^h, t \in D_j$;
* The algorithm keeps expanding $$N^{**}$$ and $$N^*$$, and transferring candidates in $$N^{**}$$ to $$N^*$$. Once the destination has been added into $$N^*$$, the shortest route is found.

## Algorithm

**Initialization:** Set the start point to $r$
* $N^*=\varnothing, N^{**}={r}$
* $A_1 = {0}, D_1=\varnothing, A_j=D_j=0$
* $k = r$

**Transfer Node:** transfer valid node $k$ from $$N^{**}$$ to $N^*$, update $D_k$
* $N^* = N^* \cup \\{k\\}$, $$N^{**} = N^{**} - \{k\}$$
* $D = g(A_k, P_k) - D_k$, $D_k = D_k \cup D$, $A_k = \varnothing$. Here excluding $D_k$ because we need a new departure time if we do not visit $k$ for the first time. The old time needs to be removed to update $D_k$. After moving $k$ into $N^*$, $A_k$ can be set to $\varnothing$ because it has already been an available point.

**Search for Available Nodes:** search for available nodes and add them into $N^{**}$, update $A_j$
* $\forall j \in N_k$, compute $A_j = A_j \cup \\{t + d_{kj}(t),t \in D\\} - D_j$. Here is the same reason as mentioned to exclude $D_j$. We always need the current arrival and departure time.
* $\bar N_k = \\{ j \in N_k, A_j \not = \varnothing\\}$ represents all reachable nodes start from $k$
* $$N^{**} = N^{**} \cup \bar N_k$$, add reachable nodes to $N^{**}$
* if $N^{**} = \varnothing$, then the algorithm is terminated because there is no feasible route. Otherwise continue to the next step

**Search for the Nodes with Shortest Time Consuming:** in all available nodes, search for the one with shortest time comsuming. If it is the destination point, then terminate. Otherwise transfer it to $N^*$ and keep loop
* $\forall j \in \bar N_k$, compute $T_j = inf\\{t, t\in A_j\\}$. Here we define $T_j$ as the infimum of time consuming when entering node $j$
* Let $$T^* = Min\{T_j, j \in N^{**}\}$$, and $$k \in N^{**}$$ be the node for which $$T_k = T^*$$. If $$k$$ is the destination node, then terminate, $$T^*$$ is the length of the shortest route. Otherwise transfer $$k$$ from from $$N^{**}$$ to $$N^*$$. **Here $k$ actually means the next valid point, instead of the $k$ mentioned in the first step.**

# Application Scenario

Up to now I have not seen any implementation of the idea. But an assumption is that the method can be used to **cut a path into pieces if the agent has to stop anywhere on the path.** For example, when multiple agents working together and there some crossovers on their paths, the crossover point can be regarded as a new node and paths of them need to be re-calculated to ensure the global optimization.


# Something Confused

This note is mostly a copy from the orginal paper *Shortest Route with Time Dependent Length of Edges and Limited Delay Possibilities in Nodes* by *J.Halpern*. I just add some note on where there can be misunderstood if you read it for the first time. But still, I am confused in the following part:

* The organism of updating $A_j$;
* Why not set $N^{**}$ to $\varnothing$ after transfer node. It looks like not only one node is added in step 3;
* It looks like $T^*$ represents the arrival time of entering the destination, why it is the length of the shortest route?
