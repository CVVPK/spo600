---
layout: post
title: Profilling with Valgrind
date: 2019-11-13
categories: Project
---

As I mentioned [before][lastpost] I couldn't get gprof to work to profile Opus. What I ended up finding out was that gprof doesn't work with shared libraries, and since Opus is meant to use its own shared libraries gprof couldn't record any time. So I had to find an alternative as I wasn't satisfied with just the perf data. Valgrind is a tool that can be used for debugging and profiling. In short it works by creating a virtual machine in which our code gets run, and it is able to record more data than perf. However, it slows down the process and what I found was that I couldn't make test runs with files that were bigger than 5-6 MB because the profiling would time out. My solution was to use one of the Opus' test vectors of ~5 MB.

Valgrind is a suite of tools, the particular tool of interest is [Callgrind]. With Callgrind we are able to record the function calls, and from the data recorded we can generate a call graph. To visualize the data we can use [Kcachegrind], which allows us to see the entire flow of our program and we can even look at the callgraph.

**INSERT CALLGRAPH HERE**

The callgraph agrees with some of the data recorded by perf. We once again see `silk_NSQ_del_dec_c` consuming most of the process' time. I initially thought I'd look into optimizing this function since it consumes so much time, but it turns out that it's a huge function (over 300 lines) and as such it may be too much for this project considering the constricted amount of time we have.

So I had to resume my search for a smaller function so that I may complete the project in time. `silk_insertion_sort_increasing` caught my eye, the name of the function would indicate that its using insertion sort which is one of the slowest sorting algorithms with a worst case of O(n^2), though in a sorted array it has a best case scenario of O(n).

It is interesting to see them using insertion sorting, and it could be the best for this use case. However, seeing that roughly 8% of execution time is being spent in this function makes me wonder if there may not be a faster way of doing it. It may just be that the data I used was the worst case scenario and maybe in average this function is expected to work in a near best case scenario.

## Next steps

I've decided to further investigate the `silk_insertion_sort_increasing` function, and find if there is a way of improving it. First I'll try to use different sorting algorithms, already looking through some of the [SILK documentation] it seems shell sort was considered at some point but it never got implemented by the Opus team. This gives me an opportunity to investigate a possible optimization, I'll continue trying to contact the developer team and discuss whether shell sort has already been tested or if there is a reason for it to not having been implemented. My goal will now be to find a way to optimize sorting on all architectures, which means making changes in plain C and not using any architecture specific intrinsics or assembler.

[lastPost]:{{site.baseurl}}{% post_url Opus:-Initial-Profiling %}
[Callgrind]: http://valgrind.org/docs/manual/cl-manual.html
[Kcachegrind]:http://kcachegrind.sourceforge.net/html/Home.html
[SILK documentation]:https://tools.ietf.org/html/draft-vos-silk-01#page-416
