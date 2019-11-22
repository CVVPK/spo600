---
layout: post
title: Sorting Algorithms
date: 2019-11-19
---

As I mentioned in my [previous post], I have chosen the function that I'm going to attempt to optimize inside the Opus reference encoder. The function I'm focusing on is called `silk_insertion_sort_increasing`. The primary job of this function is to sort a vector in-place. To do this it is currently using an insertion sort, I believe there may be a better approach, which has led me to analyzing a couple of other sorting algorithms and I'll then attempt to implement them.

There are two things that are used to analyze the efficiency of an algorithm: (1) Time Complexity and (2) Space complexity. Usually we take the worst case scenario approach to explain these complexities with the [big O notation].

## Insertion Sort

This is the current algorithm being used by the Opus encoder. It's easy to implement, however it has a O(n<sup>2</sup>) time complexity when sorting a completely unsorted array.

There is an argument to make for Insertion Sort though. In the case that it receives a fully sorted array its time complexity is O(n), or linear with the size of n. As we will see below other sorting algorithms have the same time complexity regardless of wether the initial array is already in sorted order or not.

## Shell Sort

Shell sort is a modification of Insertion sort that prioritizes sorting elements that are far apart. To do this the array is broken into smaller groups, and then we sort each group with insertion sort. Rather than using contiguous elements to create the subgroups, we use what is often called a gap. The gap indicates how far apart the elements selected are.

It may seem as if there is no point in using Shell Sort over Insertion Sort since the final step still involves the latter. However, because on each pass we produce a more sorted array the final pass will be more efficient. The gap is very important, as it will determine the final time complexity of our solution.

My first solution:
O(n^2) complexity and proved to be just as slow as the original version.

```c
 for (g = K / 2; g > 0; g /= 2)
    {
        /* Sort vector elements by value, increasing order */
        for (i = g; i < K; i++)
        {
            value = a[i];
            for (j = i; (j >= g) && (value < a[j - g]); j -= g)
            {
                a[j] = a[j - g];     /* Shift value */
                idx[j] = idx[j - g]; /* Shift index */
            }
            a[j] = value; /* Write value */
            idx[j] = i;   /* Write index */
        }
    }
```

Second attempt. Based on Knuths' algorithm (https://www.tutorialspoint.com/data_structures_algorithms/shell_sort_algorithm.htm).
Should have a base case of O(n) and a worse slightly better than O(n2) at O(n3/2)

```c

 while (g <= K / 3)
        g = g * 3 + 1;

    for (; g > 0; g = (g - 1) / 3)
    {
        /* Sort vector elements by value, increasing order */
        for (i = g; i < K; i++)
        {
            value = a[i];
            for (j = i; (j > g - 1) && (value < a[j - g]); j -= g)
            {
                a[j] = a[j - g];     /* Shift value */
                idx[j] = idx[j - g]; /* Shift index */
            }
            a[j] = value; /* Write value */
            idx[j] = i;   /* Write index */
        }
    }
```

[previous post]:
[big O notation]:
