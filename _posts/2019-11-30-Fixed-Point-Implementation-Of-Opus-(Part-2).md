---
layout: post
title: Fixed Point Implementation Of Opus (Part 2)
date: 2019-11-30
categories: Project
---

In my last [post] I mentioned that Opus has a fixed point implementation with some intrinsic optimizations, but floating point was being used by default. I have since been able to run the fixed point implementation and run some tests on it.

I tested the fixed point implementation on two different AArch64 machines: (1)Cortex-A57 & (2)Cortex-A53. Tests were conducted with a 500 MB file as input and settings were passed to the encoder to force the use of the SILK codec. Here are the results of the average total runtime:


| CPU        | Fixed Point | Floating Point |
| ---------- | ----------- | -------------- |
| Cortex-A53 | 2m38.317s   | 3m5.517s       |
| Cortex-A57 | 0m44.522s   | 0m55.711s      |

These results showed a lot of promise, there's about a 15%-20% improvement by using the fixed point implementation over the default floating point one. However, these results are only true when the encoder only uses the SILK codec. 

As I detailed in a [previous post] Opus uses two audio codecs: SILK and CELT. This means that using the fixed point implementation for the entire code base would require further testing to consider what happens when the CELT codec is used by the encoder. Moreover, the Opus encoder treats VoIP data differently to high quality audio, so these need to also be tested if I were to propose a change to the default implementation on AArch64.

When emulating the encoding of VoIP data using a fixed point implementation of the CELT codec there is still about a 4% performance increase over using floating point. However, when we emulate the use of the encoder for high quality audio the performance is impacted by using fixed point. In average the cost of using fixed point to encode high quality audio is around 2%.

The issue this presents is that while I was able to test the separate codecs, the Opus codec is meant to do the switching on the fly and also work in a hybrid mode, which is the defining characteristic of the Opus codec as it allows for high quality of speech at low bit rates. 

Unfortunately, in an average use case we'd want the highest possible audio quality, but the fixed point implementation results in slower performance, which makes changes to make the fixed point implementation the default unlikely to be accepted by the upstream. 

I thought about using the fixed point implementation only for the function I had identified (`silk_warped_autocorrelation`), but the current fixed point and floating point implementations are incompatible with each other. As far as I'm able to identify, they use vastly different approaches which means I can't just simply cast from one data type to the other and take advantage of the intrinsic optimizations that are already in the code base.

The fixed point implementation is currently recommended for embedded systems. Through the benchmarks I realized, it can be confirmed that it is indeed not the best option for current AArch64 systems. I spoke with a member of the community and learned that the fixed point implementation also yields lower quality results, which means it defeats the purpose of the Opus codec of providing a higher quality audio than low-delay codecs.

Thus, in order to use the Opus encoder for its intended purpose of encoding high quality audio data at low bit rates, the default floating point implementation is a far better option in AArch64.

The fixed point implementation of the `silk_warped_autocorrelation` function I had selected includes an intrinsic optimization. These optimizations use SIMD instructions, and according to perf and callgrind data it only takes ~7% of the total run time. Compared to the ~13% that the floating point implementation takes it is a huge improvement. It may be that most of the increase in performance in this function is due to the use of vector registers. 

Though it may be outside the time frame I had for this project, I plan to investigate the possibility of vectorising the floating point implementation of `silk_warped_autocorrelation`, by using the fixed point intrinsic optimization as a reference. That may be a way to get a useful optimization in the upstream, now that I have verified that this particular function does perform better with the fixed point implementation that is currently in the code base. 

[post]:{{site.baseurl}}{% post_url 2019-11-27-Testing-Fixed-Point-Implementation-Of-The-Opus-Encoder %}
[previous post]:{{site.baseurl}}{% post_url 2019-11-09-Opus %}