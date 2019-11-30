---
layout: post
title: Testing Fixed Point Implementation Of The Opus Encoder
date: 2019-11-27
categories: Project
---

As I mentioned on my [previous post], I tried to implement different sorting algorithms and did some more benchmarking of the Opus encoder, which led me to the conclusion that the current implementation of Insertion Sort is the best match for the use case of the encoder. From the latest profiling I did, I had identified another function that if optimized may have a bigger impact. The function is called `warped_autocorrelation_FLP`. 

This particular function takes up about 10-13% of the run time and uses a floating point implementation to calculate the [autocorrelation] of an audio signal. Digging through the code I found out that there is a fixed point implementation of this function. In addition, there are also optimizations using intrinsics.

Looking at the Opus documentation I found out that to use the fixed point implementation we simply need to define the FIXED_POINT macro at compile time. To define a macro when using make you can use:

```
make CFLAGS=-DMACRO
```

So, I attempted to compile and... It didn't work. I got multiple errors that look like this:

```
silk/float/SigProc_FLP.h:46:5: error: unknown type name ‘silk_float’; did you mean ‘silk_max’?
     silk_float          *ar,                /* I/O  AR filter to be expanded (without leading 1)                */
     ^~~~~~~~~~
     silk_max
 ```
 
 It appears some parts of the floating point implementation aren't guarded by [preprocessor directives], so they throw errors because the floating point datatype wasn't defined.

 My next step will be to look into the codebase and see if I can just add the necessary guards so that I can compile and force the encoder to use the fixed point implementation. 


[previous post]:{{site.baseurl}}{% post_url 2019-11-25-Issues-Trying-To-Optimize-The-Opus-Encoder %}
[autocorrelation]: https://pages.mtu.edu/~suits/autocorrelation.html
[preprocessor directives]: http://www.cplusplus.com/doc/tutorial/preprocessor/