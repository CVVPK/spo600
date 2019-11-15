---
layout: post
title: "Opus: Initial Profiling"
date: 2019-11-10
categories: Project
---

As mentioned [previously][opus post], I've chosen the [Opus] reference encoder as my final project for [SPO 600]. The next step is for me to look at where there are optimization possibilities. Software profiling can help with this task as it will allow me to see what instructions take most of its time and have an overview of how the application is achieving its results. With this information I intend to find possible hot spots, so that I may narrow down my efforts towards a specific area of the code base.

## Running time

The encoder was tested by running the following command 100 times on x86_64 and AArch64. File sizes tested were 10 Mb, 100 Mb and 1 Gb to check if they scaled consistently.

```
time ./opus_demo -e audio 48000 2 320000 ../100M temp.out
```

`-e` means we are running only the reference encoder. `audio` refers to the mode which could be voip, audio or lowdelay. `2` is the number of channels (Stereo = 2, Mono =1). `320000` represents 320 kb/s bit rate. `../100M` is the input file, this is just a 100Mb file with garbage data I made to test run time and nothing else. `temp.out` is the output file.

The purpose of running this command was to get an idea of run times. The average total execution time were as follows:

| File size | x86_64  | AArch64    |
| :-------- | :------ | :--------- |
| 10 Mb     | 0.456s  | 4.101s     |
| 100 Mb    | 4.132s  | 40.732s    |
| 1 Gb      | 42.002s | 6m 55.608s |

---

The running time results show AArch64 is consistently slower than x86_64 by roughly a factor of 10. We can also note that time complexity appears to be linear, which is great. However, it makes my job of finding an optimization possibility harder.

## Sampling with perf

`perf` is a tool that allows us to sample our software. By running it on the opus_demo to encode random data we can start looking at what the big time consumption functions may be. According to `perf` these seem to be the functions in which the program is spending most of its time:

```
# Overhead  Command    Shared Object     Symbol
# ........  .........  ................  .........................................
#
    12.66%  opus_demo  libopus.so.0.8.0  [.] op_pvq_search_c
    11.69%  opus_demo  libopus.so.0.8.0  [.] celt_encode_with_ec
     6.77%  opus_demo  libopus.so.0.8.0  [.] opus_fft_impl
     6.45%  opus_demo  libopus.so.0.8.0  [.] tonality_analysis.isra.0
     5.22%  opus_demo  libopus.so.0.8.0  [.] celt_pitch_xcorr_float_neon
     5.20%  opus_demo  libopus.so.0.8.0  [.] pitch_downsample
```

Running `egrep` to find where these functions are in the code returned that they all belong to the [CELT] part of the implementation. This is possibly because the encoder was run with the audio option and using a high bit rate, which forces the use of the CELT codec as it provides better results for high quality audio.

Performing another sampling, this time simulating voip data at a low bit rate, returns a different report:

```
# Overhead  Command    Shared Object     Symbol
# ........  .........  ................  ....................................................
#
    21.59%  opus_demo  libopus.so.0.8.0  [.] silk_NSQ_del_dec_c
    13.13%  opus_demo  libopus.so.0.8.0  [.] silk_warped_autocorrelation_FLP
     7.00%  opus_demo  libopus.so.0.8.0  [.] tonality_analysis.isra.0
     5.59%  opus_demo  libopus.so.0.8.0  [.] opus_encode_native
     4.97%  opus_demo  libopus.so.0.8.0  [.] silk_NLSF_del_dec_quant
     4.36%  opus_demo  libopus.so.0.8.0  [.] silk_biquad_float

```

This time, as you might derive from the function names, the [SILK] codec was being used. What looks interesting is that unlike the previous encoding this one seems to spend a whole deal more time inside a single function, 21.59% is taken by `silk_NSQ_del_dec_c`. The time difference between the two most used functions is interesting in this case, because the most time consuming function is using nearly twice the amount of time.

## Next Steps

There is further testing I'd like to perform, and I'm attempting to reach out to the community behind this project in hopes they can direct me towards any areas they know require attention. In addition to this, I'll start looking into the `silk_NSQ_del_dec_c` function as from the initial sampling done with `perf` it appears to be using a good portion of the time, it may turn out that there are possible improvements that could be done.

Further testing may include some instrumentation profiling since sampling with `perf` may have missed some function calls. Unfortunately, `gprof` is not able to get any time data and at the time of this writing I was not able to figure out why this may be. So I'm hoping someone in the community may be able to direct me towards a particular profiling tool that they use.

[Opus post]: {{site.baseurl}}{% post_url 2019-11-09-Opus %}
[Opus]: https://opus-codec.org/
[SPO 600]: https://opus-codec.org/
[SILK]: https://en.wikipedia.org/wiki/SILK
[CELT]: http://celt-codec.org/
