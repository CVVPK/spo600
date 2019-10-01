---
layout: post
title: Scaling Audio Through Software
date: 2019-09-30
categories: Lab
---

There are some cases in which we may need to modulate audio using software. Sometimes our hardware simply does not allow us to get to a low enough volume. We can achieve this by multiplying the source by a value between 0 and 1. However, this will have to be done to each sample in the file. Which means doing it, most likely, about 88,200 times per second. As we will see, for most modern hardware, this is a trivial task.

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

The first approach we are going to take a look is possibly the simplest one. We are going to multiply the sample by a volume factor. Because we are working with floating point, we will have to cast our int16 sample to a float. This operation does come at a cost, and we should expect to notice it when compared to the other approaches. 

At 5,000,000 samples this was the second slowest solution.

With:
```
Result: 94

real	0m0.526s
user	0m0.515s
sys	0m0.010s

```
Without:
```
Result: 316

real	0m0.511s
user	0m0.491s
sys	0m0.020s
```




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

This time we are going to first create an array of pre-calculated values in the range of -32768 to +32767 multiplied by the volume factor. We will then use this lookup table to find out what the output sample should be based on the index of that value on our array. Though this means more memory consumption, we might expect a faster response rate since we don't have to cast int16 to floats on each operation.

At 5,000,000 samples this was the slowest approach. 
With:
```
Result: 788

real	0m0.642s
user	0m0.620s
sys	0m0.021s

```

Without:

```
Result: 316

real	0m0.512s
user	0m0.480s
sys	0m0.030s
```

## Bit-shifting and integer multiplication

```c
  int16_t vol= 0.75 * 0b100000000;
        for (x = 0; x < SAMPLES; x++) {
                data[x] = data[x] * vol >> 8;
        }
```

Finally we're going to take a look at an approach that I believe to be the most efficient. Firstly, we will convert our floating point volume value to a fixed point one. We're going to use 256 (0b100000000 in binary). Then, we're going to bit-shift 8 bits to the right to get the new sample at the desired volume.

Surprisingly this was just barely faster than the first approach that used floating point.
With:

```
Result: 873

real	0m0.522s
user	0m0.491s
sys	0m0.030s
```

Without:

```
Result: 316

real	0m0.512s
user	0m0.481s
sys	0m0.029s
```

[Lab 4]: https://wiki.cdot.senecacollege.ca/wiki/SPO600_Algorithm_Selection_Lab

