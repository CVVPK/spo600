---
layout: post
title: Issues Trying To Optimize The Opus Encoder
date: 2019-11-25
categories: Project
---

So after realizing [Shell Sort] didn't provide any performance improvements over the insertion sort implementation in the Opus encoder, I decided to take a pause and make sure the path I was following was correct.

I've found some issues with the strategy that I had decided to follow and why trying to optimize the sorting procedure within the Opus encoder may not be the best idea.

Below is the implementation of insertion sort that is currently being used:

```c
    for( i = 1; i < K; i++ ) {
        value = a[ i ];
        for( j = i - 1; ( j >= 0 ) && ( value < a[ j ] ); j-- ) {
            a[ j + 1 ]   = a[ j ];       /* Shift value */
            idx[ j + 1 ] = idx[ j ];     /* Shift index */
        }
        a[ j + 1 ]   = value;   /* Write value */
        idx[ j + 1 ] = i;       /* Write index */
    }

    /* If less than L values are asked for, check the remaining values, */
    /* but only spend CPU to ensure that the K first values are correct */
    for( i = K; i < L; i++ ) {
        value = a[ i ];
        if( value < a[ K - 1 ] ) {
            for( j = K - 2; ( j >= 0 ) && ( value < a[ j ] ); j-- ) {
                a[ j + 1 ]   = a[ j ];       /* Shift value */
                idx[ j + 1 ] = idx[ j ];     /* Shift index */
            }
            a[ j + 1 ]   = value;   /* Write value */
            idx[ j + 1 ] = i;       /* Write index */
        }
    }
```

What this code does is correctly sort the first `K` elements inside of an array of length `L`. To do this it first sorts the initial `K` elements with the first `for loop` and then the second loop goes over the remaining `K-L` elements and compares them to the already sorted `K` elements. The result is a highly optimized procedure in which time isn't wasted moving elements that are irrelevant (elements that come after the first `K` elements).

Something I should have initially looked at was how big the arrays being sorted were. I suspected this function was being used to sort very small arrays because that's where insertion sort shines. The values turn out to consistently be between 1 and 16 for `K` and 16 to 32 for `L`, so the for loops are doing very small iterations.

I had assumed there could be a possible improvement that could be done, the first thing I wanted to try was implementing shell sort. As it was noted in my previous [post][Shell Sort], using shell sort didn't make any impact to the performance. The overhead introduced by other algorithms may not be worth it. I thought of trying to use an O(n log n) solution before deciding to change my approach to optimizing the Opus encoder, but there was something I wanted to take a look before. 

The next thing I did was literally remove all the sorting logic and replaced it with some dummy code just to avoid getting compiler warnings. This of course meant the encoder would no longer do its job, but what I wanted to see was what impact it would have to not sort any more.

This is the result of encoding a 500 MB file without doing any changes to the sorting implementation:

```
real	3m5.769s
user	3m3.899s
sys	0m1.332s
```
And here is what happens when sorting is removed:

```
real	3m5.623s
user	3m3.802s
sys	0m1.265s
```

A rounding error, it really changed nothing in terms of performance. Interesting, since when I did my [initial profiling using Callgrind] this function showed up with a significant amount of run time. I've redone the profiling with Callgrind without sorting and got the same results I had before, which has led me to stop trusting them. It is possible that because the sorting function runs so fast, but gets called thousands of times successively, Callgrind is accumulating all these calls and that's why the function appears to take so much time.

At this point it has become quite clear that there is no real optimization I could do to the sorting algorithm that would improve the performance of the Opus encoder. The current implementation is extremely optimized for this particular use case, and the added overhead of other algorithms like shell sort, heap sort or quick sort nullifies the reduced time complexity they offer. 

So, in hopes of being able to make a change that actually presents a performance improvement I'll be looking at a different function. 

This time I've learned how to do a quick check that helps me verify the data I get from perf and Callgrind (remove all the logic from the function and crosscheck run time). The function I've found is `warped_autocorrelation_FLP`. According to Callgrind this function takes around 10% of the run time, perf recorded around 13%. To verify this data I did what I should have done much earlier in the past and removed all the logic from the function, the result was a difference of around 20 seconds when encoding a 500 MB file, which is about 13% of 3m5s (the regular run time).

Thus, I've confirmed that the data from perf and Callgrind about this particular function is accurate. What this means for me is that I first need to figure out what `warped_autocorrelation_FLP` does, and then make a new strategy as this is a very different function from my first choice.

[Shell Sort]:{{site.baseurl}}{% post_url 2019-11-19-Shell-Sort %}
[initial profiling using Callgrind]: {{site.baseurl}}{% post_url 2019-11-13-Profiling-The-Opus-Encoder-with-Callgrind %}