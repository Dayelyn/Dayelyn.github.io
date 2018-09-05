---
layout: post
title:  "A* with the heuristic function of FastMap for path planning"
date:   2018-09-04 10:25:51 +0800
categories: Literature_Study
tags: Path_Planning A* FastMap
description: The second literature study. Introduce a modified FastMap heuristic function for A* algorithm for path planning. You can get a overall on the FastMap algorithm and how to modify it to adapt to A*. The detail A* needs to added in the future.
---

<script type="text/javascript" async src="//cdn.bootcss.com/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# Article Title
The FastMap Algorithm for Shortest Path Computations.
By Liron Cohen etc.

# Forewords

Although I have some unsolved problems in the first paper, I still move to the second one. A project needs to set a goal or milestone, otherwise it will be endless. So as to the literature study. I guess I can ask my mentor for help after they come back from their tough project :)

# Abstract

In this paper, a given edge-weighted undirected graph is projected to a $k$-dimension Euclidean space by a modified FastMap algorithm. Then, the shortest path between any two nodes can be computed by $A^*$ search with Euclidean distance as the heuristic function.

In the note I am going to start with the FastMap algorithm and the $A^*$ search. Then the modified algorithm will be introduced.

# FastMap
FastMap is a dimension reduction method with short time comsumption and mostly used in data mining. See more on the [original paper](http://www.cs.cmu.edu/~christos/PUBLICATIONS.OLDER/sigmod95.ps.gz). It can map any abstract objects into a $k$-dimension euclidean space with keeping their geometric relations at most, where $k$ can be specified by users.

The basic idea of the FastMap is to look for the furthest pariwise node as a coordinate in every iteration, map all other nodes on the coordinate, then map all nodes to a hyperplane that is perpendicular to the coordinate and enter the next iteration. With the recursive step, all nodes will be mapped into a $k$-dimension space with orthogonal coordinates.

Since we are going to use the FastMap with $A^*$ in a given edge-weighted undirected graph, all notations in the following part of the article are base on the gragh theory.

The work flow of the FastMap is as follows:

## Step 1: Find the Furthest Pairwise Node

In the first iteration, we need to find the furthest pair of nodes as the coordinate of the first dimension. 

![Imgur](https://i.imgur.com/u2nnAVZ.png)

Algorithm 1 in the figure above shows the details of how to find the furthest pair of nodes. Here the algorithm has to be repeated several times to have the final result. The reason of repeatition is to reduce the computation complexity. One important thing is that the **distance** is based on user setting. In the next part we will set a special function to calculate the distance.

## Step 2: Project All Other Nodes on the Generated Coordinate

According to **Step 1**, we have a coordinate alongside $O_a$ and $O_b$. Then as shown in figure 1, we can project all other node on the coordinate with the help of cosine law:

![Imgur](https://i.imgur.com/rWyGBC8.png)

For a arbitrary node $O_i$, the coordinate $x_i$ projected on axis $(O_a, O_b)$ can be calculate with cosine law. Particularly, $x_a = 0, x_b = d_{ab}$.

## Step 3: Find a Hyperplane that is Perpendicular to the Current Coordinate, Project All Nodes on the Plane

Up to now we have already reduce all nodes into a 1-d space. If you would like to analyze the data in a higher dimension, we need to project nodes in another dimension, which is perpendicular to the current dimension, as shown in figure 2.

![Imgur](https://i.imgur.com/mglg5iL.png)

We can see that if we project all nodes in the new dimension, $x_a = x_b = 0$ due to the perpendicularity. Since now all nodes are on a 2-d Euclidean space, we can use the $L_2$ distance as the distance function (the formula is given in 3-d Euclidean space).

## Step 4: For All Nodes on the Hyperplane, Go to Step 1

Still, we would like to find the furthest pairwise node to project all nodes on the coordinate established by them. Now we can see that users can specify any $k$ for $k$-dimension to reprensent the data.

# $A^*$

I would prefer adding it later, since the paper is mainly focused on the modified FastMap.

# Modified FastMap

In this paper, authors make two modifications on the FastMap algorithm to adapt to the path searching problem.

## Distance Function for Finding the Furtherest Node

As mentioned above, in the first step in the FastMap we need to calculate the "furthest" pair of nodes. The distance function is specified by users. In the modified version, since we focus on a edge-weighted undirected graph, a shortest path tree rooted at a random node $n_a$ is construct. Then we search for the deepest node in the tree as the "furthest" node and the depth is $d_ab$.

After the establishment of the coordinate, other parameters referring to distance in figure 1 can be attained by the same way, which means $d_{ai}, d_{ib}$ are all calculated by the shortest path tree rooted at $n_a$.

## Distance Function on the Coordinate and the Hyperplane

Recall that we use the $L_2$ distance in Step 2 and 3. The author of the paper proves that it has to be changed to $L_1$ distance in order to keep **admissibility and consistency**.

Then the distance on the $K^{th}$ coordinate needs to be changed to:

$$
[p_i]_K = \frac{(d_{ai}-d_{ib}+d_{ab})}{2}
$$

and the distance on the next hyperplane needs to be changed to:

$$
D_{new}(O_i, O_j) = D_{old}(O_i,O_j) - |[p_i]_K - [p_j]_K|
$$

specifically, in our case the $D_{new}$ and $D_{old}$ refers to weight in the graph.

The modified FastMap algorithm can be seen in the following figure:

![Imgur](https://i.imgur.com/Ys6BRPj.png)

# Application Scenario

Basically this is an optimization for $A^*$ heuristic function. It is well known that a better heuristic function can lead to a more optimal path. This method probably can output a better plan than other heuristic function.

# Something Confused
I guess the author has demonstrated the algorithm clearly in the paper, but I am not sure about how important the admissibility and consistency are to the $A^*$. 