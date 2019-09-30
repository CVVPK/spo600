---
layout: post
title: Audio Data
date: 2019-09-30
categories: other
---

You are probably familiar with at least some of the formats used to store audio files: mp3, aac, wma, etc. You might also know that data in a computer is always stored in binary. But, how do we go from a sound wave to a binary representation of data.

The answer is by sampling the waveform. Sampling refers to the conversion of an analogue signal into a digital one. To sample sound we measure the waveform at specific intervals and then store each measurement as a binary value.

Since we are taking measurements at regular intervals, some of the data in between measurements will be lost. To prevent loosing too much data we ensure that our sampling rate is high enough. The higher the sampling rate, the better the reproduction of data will be. However, you'll also end up with more data and more data means a higher file size.

The most common sampling rate today is 44.1 kHz, which is what is used by the CD music industry. Which perfectly allows us to reproduce sound inside the human hearing range of 20 Hz - 20 kHz.

## Why 44.1 kHz?

The 44.1 kHz was determined using the [Nyquist Sampling Theorem][nyquist theorem], which states that a waveform can be perfectly reconstructed if it's sampled twice as fast as its highest frequency.

Since human hearing peaks out at about 20 kHz in the best of cases (a middle-aged adult may only hear up to 14 kHz) we can safely destroy all the data above this range without consequence. So by the Nyquist Sampling Theorem, if the highest frequency we need to represent is 20 kHz that means sampling rate needs to be at least 40 kHz.

The problem with a 40 kHz sampling rate comes from using a [low pass filter][low pass filter], which is what is used to filter out the frequencies above the 20 kHz range. A low pass filter uses a [transition band][transition band] to attenuate the frequencies just before the cutoff point. This transition band even if unwanted is not only necessary but also quite obvious. 

The transition band can be made wider in order to make the cutoff less obvious. So, a 2.05 kHz transition band is used as an anti-aliasing filter. By applying the Nyquist Sampling Theorem it results in 22.05 kHz * 2 = 44.1 kHz.

## How big are audio files?

A sampling rate of 44.1 kHz means 44,100 samples per second at 16 bit per sample that comes up to 705,600 bits per second of data. But, that is only one channel of data and stereo audio requires two channels. Therefore, per second of audio data at 44.1 kHz we end up with 1,411,200 bits per second or 176.4 KB. A 4 minute song results in 42.336 MB of data. That's huge!

Just imagine your data bill if all you streamed was uncompressed audio. My phone says that over the past week I have used spotify for nearly 24 hours, if you do the math that would have been just over 15 GB of data. 

Most music we listened to is not from a raw audio file, however. You might be familiar with files that are about 1-2 MB per minute of audio, which is partly the reason why we don't go broke from streaming music. The reason of that is data compression.

## Audio Compression

There are two basic techniques of data compression: lossless and lossy. Lossless compression means that we can go back to the original with no loss of data, this also means a bigger file size. On the other hand, lossy compression does allow for the loss of some of the data but yields a smaller file. 

Lossy compression has come a long way and thanks to advances in [psychoacoustics][psychoacoustics] we can now enjoy music with no perceivable data loss at a very low memory cost. However, there are still those who claim that nothing will ever replace their [FLAC][flac] files.


[nyquist theorem]: https://en.wikipedia.org/wiki/Nyquist%E2%80%93Shannon_sampling_theorem
[low pass filter]: http://www.learningaboutelectronics.com/Articles/Low-pass-filter.php
[transition band]: https://en.wikipedia.org/wiki/Transition_band
[psychoacoustics]: https://en.wikipedia.org/wiki/Psychoacoustics
[flac]: https://en.wikipedia.org/wiki/FLAC