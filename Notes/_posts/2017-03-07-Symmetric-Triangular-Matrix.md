---
layout: post
title: Symmetric Triangular Matrix
---
## Introduction
If you have worked with graphs you've probably made use of an <a href="https://en.wikipedia.org/wiki/Adjacency_matrix">adjacency matrix</a>. But if your graph is undirected, you can notice that the element [i][j] is equal to [j][i]. Something similar happens when doing DFA Minimization. Unfortunately, this way we are "wasting" half of the memory and it doesn't sound good.

So what we would like to have is a data structure that works exactly the same way but using half of the memory. We can do it easily just with a bit of calculus. By the way, in a graph we don't usually need the diagonal but in my example I'm going to consider it to make it simpler.<!--more-->

<br />
## Lower and Upper
We are going to use a linear array to keep a fast access time and we will calculate the index manually. The triangular matrix can be lower or upper triangular:

<img src="https://fylux.github.io/public/img/TMatrix.jpg" width="100%">

We are going to use the lower triangular and I'll tell you why later.

The size of the array instead of N<sup>2</sup> will be *1 + 2 + 3 + ... + N = (N<sup>2</sup> + N)/2* an <a href="https://en.wikipedia.org/wiki/Arithmetic_progression">arithmetic progression</a>, isn't it?

Now to obtain the position [i,j] in the matrix, we will do the following. Because it is lower triangular, if j is greater than i, then i=j and j=i.
So the index in the linear array is:

$$
index = \frac{i^2+i}{2} + j
$$

It doesn't require much explanation. Essentially we calculate the corresponding i position using the arithmetic progression and then we add j.

If you had chosen an upper triangular matrix, the formula would be a little more complex because you would need to calculate something like *n + n-1 + n-2 ... n-i* and I prefer to keep things as simple as possible.

<br />
## Conclusion
I just want to add that using half of the memory is always a great advantage. It would be interesting to study if there is a little overhead because of using a function to calculate the position. Definitely, it will be damn little the time used in calculus, but if the bottleneck of your code is in array indexing and you don't care about using double of memory for the matrix, you may want to study if you can obtain any advantage using a common adjacency matrix. 

<br />
## Implementation
I've developed a simple implementation in C++ to play with it.<br/><a href="https://github.com/fylux/TriangularMatrix">Triangular Matrix Implementation in C++</a>