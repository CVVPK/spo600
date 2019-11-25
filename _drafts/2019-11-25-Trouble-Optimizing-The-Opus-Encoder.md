---
layout: post
title: Trouble Optimizing The Opus Encoder
date: 2019-11-25
---

I decided to take a pause and make sure I'm understanding the code properly. Turns out I may have made a mistake assuming that the loop I was changing was the only responsible for the sorting. Below is the original sorting algorithm being used at the moment.

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
What I was doing previously was changing the 
