---
layout: post
title: Memory Alignment and Performance
---
## Introduction
Memory alignment is not a hot topic. Usually, we don't worry about it and very few people will bother to know if the pointer returned by <a href="http://www.cplusplus.com/reference/cstdlib/malloc/">*malloc*</a> is 16 or 64 bytes aligned. However, I suspect that memory alignment may affect the performance someway in <a href="https://en.wikipedia.org/wiki/X86-64">x86-64</a> processors.


In this article I want to analyze if memory alignment has effects on performance. 

<br/>
## Previous Work
Although this has been partially studied I'll try to fill some gaps. On the one hand, <a href="http://lemire.me/blog/2012/05/31/data-alignment-for-speed-myth-or-reality/">Lemire D.</a> covered the performance when iterating all the elements of an array. However, this case is not that interesting because if you load all the elements the number of cache misses at the end will be almost the same. On the other hand <a href="https://github.com/lemire/Code-used-on-Daniel-Lemire-s-blog/tree/master/2012/05/31">Gauthier L.</a> covered a specific access pattern which showed better performance with aligned access.

However, this approaches didn't considered the alignment of the memory allocated but the way data is accessed.


Therefore, I will focus on analyzing if the alignment of the memory allocated affects the performance when doing random access. I won't force the unalignment; instead I will use memory allocators that only guarantee some level of alignment.<!--more-->

<br/>
## Random Access
Before continuing, I'll explain why random access could have drawbacks.


Let's suppose that cache lines have a size of 64 bytes. And you have an element of that size. Thus if the data is 64 bytes aligned the element will perfectly fit in a cache line. 

<center>
<img src="https://fylux.github.io/public/img/alignment/aligned.png" width="65%">
</center>

However, if doesn't fit in a cache line it will be stored between two lines. Therefore we will need to load two blocks of memory instead of one to use the element.

<center>
<img src="https://fylux.github.io/public/img/alignment/unaligned.png" width="65%"></center>

Furthermore, <a href="https://en.wikipedia.org/wiki/SIMD">SIMD</a> can't work with very low memory alignment, so programs which rely on these instructions may be affected.

<br/>
## Experiment
For this analysis we will use the function <a href="http://en.cppreference.com/w/c/memory/aligned_alloc">*aligned_alloc*</a> that appeared in C++11. This function allows us to choose how much memory alignment is required for the allocated memory.


To test the performance we will allocate arrays with different memory alignments of 1,000,000 elements and then we will load 1% of those elements.


The code used for the analysis can be found <a href="https://github.com/fylux/fylux.github.io/tree/master/public/code/alignment">here</a>. 

<br/>
## Results
There are a few things that I have to point out before showing the results. The first is that there are big variations between test executions, so I can't ensure as many things as I would like. Thus, I'm only going to show what I am sure about after many different approaches.


The first main result is that I haven't seen serious differences while changing the size of the element that we want to load. Actually, using 2 or 4-byte alignment I've notice a small growth in execution time, while I haven't notice differences using more restrictive memory alignment.


Finally, after comparing the execution time for different memory alignments using element sizes between 32 and 96 bytes I've obtained the following graph:

<img src="https://fylux.github.io/public/img/alignment/graph.png" width="100%">

This shows that using very low memory alignment can affect the performance of our program. Nevertheless, using very restrictive memory alignment doesn't worth because I've seen any serious difference between memory alignments greater than 16 bytes.

<br/>
## Conclusion
I've found that using very low memory alignment can be harmful for performance. However, I expected to see more results related with the size of the elements. Perhaps prefetching is solving this problem. Furthermore, I guess that vectorization may have something to do with these results. In any case, it should be studied in more depth.
