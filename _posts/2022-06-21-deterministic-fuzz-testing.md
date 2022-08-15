---
toc: true
layout: post
description: Make DeepState fuzz testing with Rcpp, deterministic. Solution implemented in the pull request \#6
badges: true
categories: [DeepState, Rcpp]
comments: false
title: Deterministic fuzz testing with DeepState and Rcpp
---

## Motivation
While searching for a way to perform non deterministic fuzz testing (using some kind of seeds) to be used in the workflow for the [pull request #6](https://github.com/FabrizioSandri/RcppDeepState/pull/6), I discovered that the seed would not work if the RInside declaration is made inside the `TEST` macro of the test Harness.
Let's have a look at an example. 

## Example
Recall to the example of the previous post, where we wanted to perform fuzz testing on a simple Rcpp function that takes an integer vector as input and returns the first element. 
Let's say we want to discover a seed that will allow us to replicate the DeepState inputs throughout several executions, in a way that the execution will be deterministic. Let's add a `cout` at line 36 in order to observe the inputs generated by DeepState.
```c++
#include <iostream>
#include <Rcpp.h>
#include <RInside.h>

#include <deepstate/DeepState.hpp>
using namespace deepstate;

// [[Rcpp::export]]
int getFirstElement(Rcpp::IntegerVector v) {
    
    if (v.size() != 0){
        return v[0];
    }

    return -1;
}

// random IntegerVector generation procedure
Rcpp::IntegerVector randomIntegerVector(int maxSize){

    int size = DeepState_IntInRange(1,maxSize);
    Rcpp::IntegerVector vec(size);

    for (int i=0; i<size; i++){
        vec[i] = DeepState_IntInRange(0,1000);
    }
  
    return vec;
}


TEST(Unit, name){
    static RInside R;
    
    Rcpp::IntegerVector randomVec = randomIntegerVector(20);
    std::cout << randomVec << std::endl;

    getFirstElement(randomVec);
}
``` 

The result will change across several executions regardless of whether this harness is executed using the `--seed=1` argument.
The first ten lines of output from two distinct runs are shown below, and it appears that the seed option has no influence on the produced input. 

```bash
$ ./getFirstElement --fuzz --timeout=1 --seed=1 | head
INFO: Starting fuzzing
WARNING: No test specified, defaulting to first test defined (Unit_name)
808 622 869 877 455 672 774 844
240 910 915 474 182 907 28 1000 921 295
168 356 675
825 344 511 36 735 784 573 715 622 338 351 542
750 202 746 817 227 9 588 973 14 4 194 627 987 249 578 315 275
415 918 1 777 966 69 806 69 559 978 421 300
867 214 429 872 755 856 345 993 722
942 378 593 210 780 393 760 430 106 400 846 144
767 669 263 278 872 625
265 58 203 89 832 969 144 952 266 192 975 327 386 816 946 35

$ ./getFirstElement --fuzz --timeout=1 --seed=1 | head
INFO: Starting fuzzing
WARNING: No test specified, defaulting to first test defined (Unit_name)
657 470 974 648 599 658 812 515 388 162 698 87 20 672 429
355 22 104 1000 742 779 948 157 630 840 336 616 666 220 185 76 965 220 18
531 484 232 85
76 959 388 473 630 390 997 619 474 386 449 650 255 520 717 909 775 783 249 75
988 668
422
595
767 182 417 60 691 783 972 996 740 220 732 19 390 253 704 89 329 815
520 122 600 111 27 796 541 591 778 352 673 205 195 526 73
318 380 155 221 972 540 668 985 462 953 175
```

## Solution
This peculiar behavior, I discovered, is caused by the RInside statement within the `TEST` macro. The seed will work if the above program is modified by shifting the RInside declaration to the beginning and deleting the `static` keyword. 
```c++
#include <iostream>
#include <Rcpp.h>
#include <RInside.h>

#include <deepstate/DeepState.hpp>
using namespace deepstate;

RInside Rinstance;

// [[Rcpp::export]]
int getFirstElement(Rcpp::IntegerVector v) {
    
    if (v.size() != 0){
        return v[0];
    }

    return -1;
}

// random IntegerVector generation procedure
Rcpp::IntegerVector randomIntegerVector(int maxSize){

    int size = DeepState_IntInRange(1,maxSize);
    Rcpp::IntegerVector vec(size);

    for (int i=0; i<size; i++){
        vec[i] = DeepState_IntInRange(0,1000);
    }
  
    return vec;
}


TEST(Unit, name){
    
    Rcpp::IntegerVector randomVec = randomIntegerVector(20);
    std::cout << randomVec << std::endl;

    getFirstElement(randomVec);
}
``` 

The output will now remain the same over multiple executions
```bash
$ ./getFirstElement --fuzz --timeout=1 --seed=1 | head
INFO: Starting fuzzing
WARNING: No test specified, defaulting to first test defined (Unit_name)
45 876 469 501 681 966 606 500 505 917 974 278
278 134
30 89 629 63 662 549 902 171 509 314 370 622 287 508
904 960 320 971 431 402 436 155
610 425 643 826 670 136 877 558 789 66 681 315 4
29 888 577 658 499 191 348 959 539 740 712 152 704 460 449 727
889 811 54 456 168 891 196 561 467 544 254 942 911 256 663 578 517 845
159 167 671 342 887 546 605 607 354 881 361 545 865 588 312
649 123
750 729 474 863 43 482

$ ./getFirstElement --fuzz --timeout=1 --seed=1 | head
INFO: Starting fuzzing
WARNING: No test specified, defaulting to first test defined (Unit_name)
45 876 469 501 681 966 606 500 505 917 974 278
278 134
30 89 629 63 662 549 902 171 509 314 370 622 287 508
904 960 320 971 431 402 436 155
610 425 643 826 670 136 877 558 789 66 681 315 4
29 888 577 658 499 191 348 959 539 740 712 152 704 460 449 727
889 811 54 456 168 891 196 561 467 544 254 942 911 256 663 578 517 845
159 167 671 342 887 546 605 607 354 881 361 545 865 588 312
649 123
750 729 474 863 43 482
```

This problem is resolved in the [pull request #6](https://github.com/FabrizioSandri/RcppDeepState/pull/6).