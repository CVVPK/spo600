---
layout: post
title: Scaling Audio Through Software
date: 2019-10-02
categories: Lab
---

There are some cases in which we may need to modulate audio using software. Sometimes our hardware simply does not allow us to get to a low enough volume. We can achieve this by using software to reduce the value of each sample from the source. Since the operation needs to be done on each time we can usually expect to make it about 88,200 times per second. As we will see, for most modern hardware, this is a trivial task.

We're going to explore three different approaches to solve this problem:

- [Floating point scaling](#floating-point-scaling)
- [Pre-calculated lookup table](#pre-calculated-lookup-table)
- [Bit-shifting and integer multiplication](#bit-shifting-and-integer-multiplication)

All work detailed here was done as part of [Lab 4].

## Floating point scaling

```c
static inline int16_t scale_sample(int16_t sample, float volume_factor) {
        return (int16_t) (volume_factor * (float) sample);
}
```

The first approach we are going to take a look is possibly the simplest one. We are going to multiply the sample by a volume factor. Because we are working with floating point, we will have to cast our int16 sample to a float. Casting to a float and doing floating point math should not be taken lightly.

We tested this solution with 5,000,000 samples in an AArch64 machine, the operation took a mere 0.015 seconds. However, this was the second slowest solution of the three we tested.

## Pre-calculated lookup table

```c
static inline int16_t scale_sample(int16_t sample, float volume_factor, int16_t* table) {
        return table[sample + 32768];
}

static int16_t* lookup_table(unsigned int min, unsigned int max, float volume_factor){

        int16_t* table;
        unsigned int size = min+max+1;
        table = (int16_t*) calloc(size, sizeof(int16_t));

        for(int x = 0; x<size;x++){
                table[x] = (x-min)*volume_factor;
        }

        return table;
}
```

This time we are going to first create an array of pre-calculated values, using `lookup_table()`, in the range of -32768 to +32767 multiplied by the volume factor. We will then use this lookup table to find out what the output sample should be based on the index of that value on our array.

As you might expect, creating a lookup table with all 5,000,000 samples takes its toll on the system. Again, this was tested in an AArch64 machine. We got an average of 0.12 seconds to realize the scaling of 5,000,000 samples.

## Bit-shifting and integer multiplication

```c
  int16_t vol= 0.75 * 0b100000000;
    for (x = 0; x < SAMPLES; x++) {
            data[x] = data[x] * vol >> 8;
    }
```

Finally we're going to take a look at an approach that I believe to be the most efficient. Firstly, we will convert our floating point volume value to a fixed point one. We're going to use 256 (0b100000000 in binary). Then, we're going to bit-shift 8 bits to the right to get the new sample at the desired volume.

At an average 0.01 seconds to process 5,000,000 samples this solution was marginally better than using floating point math. We could probably see the advantage of using this approach with a greater sample size.

[Lab 4]: https://wiki.cdot.senecacollege.ca/wiki/SPO600_Algorithm_Selection_Lab

