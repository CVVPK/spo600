---
layout: post
title: Shell Sort
date: 2019-11-19
---

As I mentioned in my [previous post], I have chosen the function that I'm going to attempt to optimize inside the Opus reference encoder. The function I'm focusing on is called `silk_insertion_sort_increasing`. The primary job of this function is to sort a vector in-place. To do this it is currently using an insertion sort, I believe there may be a better approach, which has led me to analyzing a couple of other sorting algorithms and I'll then attempt to implement them.

There are two things that are used to analyze the efficiency of an algorithm: (1) Time Complexity and (2) Space complexity. Usually we take the worst case scenario approach to explain these complexities with the [big O notation].

Currently the  Opus encoder uses Insertion Sort. This algorithm has a trivial implementation, however it has a O(n<sup>2</sup>) time complexity when sorting a completely unsorted array.

There is an argument to make for Insertion Sort though. In the case that it receives a fully sorted array its time complexity is Ω(n). In addition to this, Insertion Sort tends to be really fast sorting small arrays since it doesn't add any overhead.

## Shell Sort

Shell sort is a modification of Insertion sort that prioritizes sorting elements that are far apart. To do this the array is broken into smaller groups, and then we sort each group with insertion sort. Rather than using contiguous elements to create the subgroups, we use a gap to create these subgroups. The gap indicates how far apart the elements selected are.

It may seem as if there is no point in using Shell Sort over Insertion Sort since the final step still involves the latter. However, because on each pass we produce a more sorted array the final pass will be more efficient. Determining the appropriate gap is very important, as it will determine the final time complexity of our solution.

My first implementation of shell sort is the following:

```c
 for (g = K / 2; g > 0; g /= 2)
    {
        
        for (i = g; i < K; i++)
        {
            value = a[i];
            for (j = i; (j >= g) && (value < a[j - g]); j -= g)
            {
                a[j] = a[j - g];     
                idx[j] = idx[j - g]; 
            }
            a[j] = value; 
            idx[j] = i;   
        }
    }
```
This results in a Θ(n<sup>2</sup>) time complexity which is not exactly what we wanted, and as expected had no effects in the running time of the opus encoder. There are [other ways to calculate the gap], which led me to my next solution:

```c

 while (g <= K / 3){
        g = g * 3 + 1;
 }
    for (; g > 0; g = (g - 1) / 3)
    {
        for (i = g; i < K; i++)
        {
            value = a[i];
            for (j = i; (j > g - 1) && (value < a[j - g]); j -= g)
            {
                a[j] = a[j - g];     
                idx[j] = idx[j - g]; 
            }
            a[j] = value; 
            idx[j] = i;   
        }
    }
```
This is based on Knuth's formula, it has Θ(n<sup>3/2</sup>) complexity which is better than the previous solution. However, calculating the initial value of the gap adds an overhead and the result is that it runs in about the same time as the original Insertion Sort.

We may be dealing with arrays that are too small for Shell Sort to provide any significant improvements. This makes me wonder whether other sorting algorithms may actually not do much in terms of performance, since most of them will add some overhead. However, now that I have discarded shell sort as a solution my next step still is to try a O(n log n) algorithm.


[previous post]:{{site.baseurl}}{% post_url 2019-11-13-Profiling-The-Opus-Encoder-with-Callgrind %}
[big O notation]: https://en.wikipedia.org/wiki/Big_O_notation
[other ways to calculate the gap]: https://en.wikipedia.org/wiki/Shellsort#Gap_sequences
