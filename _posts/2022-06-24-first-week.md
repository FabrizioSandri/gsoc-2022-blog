---
toc: true
layout: post
description: Pull requests and changes made in the Community Bonding Period and in the first week
badges: true
categories: [summary]
comments: true
title: First changes to RcppDeepState
---

## Introduction
This post is meant to be a reference for the work I've done in the Community Bonding Period and the first coding period's first week.


## Weeks 1-3 (Community Bonding Period)
I spent this three weeks of Community Bonding Period interacting with my mentors, familiarizing myself with the tools and working on high priority tasks. 

Using this blog as a way to familiarize myself with the tools, I started working with DeepState, showing some basic examples and moving step by step with more complex one. Then I gave a brief introduction to Valgrind and showed how it might be used in conjunction with DeepState to produce a sophisticated fuzz testing toolbox.
When these three powerful tools —Valgrind, DeepState, and Rcpp— were integrated, I introduced Rcpp and finally RcppDeepState. 

### Reference
Blog posts:
* [Sample DeepState fuzz test](https://fabriziosandri.github.io/gsoc-2022-blog/deepstate/fuzz/c++/2022/05/17/deepstate-sample.html)
* [DeepState introduction](https://fabriziosandri.github.io/gsoc-2022-blog/deepstate/fuzz/c++/2022/05/25/about-deepstate.html)
* [Advanced fuzz testing with DeepState and Valgrind](https://fabriziosandri.github.io/gsoc-2022-blog/deepstate/fuzz/c++/valgrind/2022/05/27/advanced-deepstate.html)
* [Introduction to RcppDeepState](https://fabriziosandri.github.io/gsoc-2022-blog/rcppdeepstate/fuzz/r/2022/05/31/rcppdeepstate-introduction.html)
* [Rcpp package fuzz testing with RcppDeepState](https://fabriziosandri.github.io/gsoc-2022-blog/rcppdeepstate/fuzz/r/c++/2022/06/07/rcppdeepstate-automatic-fuzz-testing-copy.html)


Pull requests:
* [Makefile generation fix and improvements](https://github.com/FabrizioSandri/RcppDeepState/pull/1)
* [Override default Makevars](https://github.com/FabrizioSandri/RcppDeepState/pull/3)
* [Harness creation improvements](https://github.com/FabrizioSandri/RcppDeepState/pull/5)

Issues:
* [Segmentation fault not catched](https://github.com/FabrizioSandri/RcppDeepState/issues/2)
* [Fuzzing functions with Rcpp parameters](https://github.com/FabrizioSandri/RcppDeepState/issues/4)

External issues:
* [Unexpected Segmentation Fault](https://github.com/RcppCore/Rcpp/issues/1221)

## Week 4
I have spent the most of this first week focusing on some important tasks that I discovered during the Community Bonding Period.
Everything specifically began when I discovered that debug symbols were not present by default on some systems. This led to the creation of the Makevars file inside the `src` directory of each package, before `R CMD INSTALL` is run. Based on this I created a custom workflow to check if debug symbols are included by default or not on the system. I discovered several bugs in the RcppDeepState library while developing this workflow and was able to effectively fix them. 

A detailed description of the changes and problems I encountered in this first week can be found in [pull request #6](https://github.com/FabrizioSandri/RcppDeepState/pull/6). I report some of them here:
1. solved the problem that on some platforms debug symbols are not included by default;
2. solved some problems with `RInside`. In detail I moved the definition of `RInside` outside of the `TEST` function in order to preserve the seeds and to avoid the `R is already initialized` error;
3. implemented the possibility to add a seed argument to the `deepstate_harness_compile_run` function;
4. solved the problem of the Segmentation fault that prevented DeepState to create the output files;


### Reference
Blog posts:
* [Debugging Rcpp outside of R](https://fabriziosandri.github.io/gsoc-2022-blog/rcpp/debug/r/c++/valgrind/2022/06/10/rcpp-debugging.html)
* [Deterministic fuzz testing with DeepState and Rcpp](https://fabriziosandri.github.io/gsoc-2022-blog/deepstate/rcpp/c++/r/2022/06/21/deterministic-fuzz-testing.html)

Pull requests:
* [Debug symbols tests](https://github.com/FabrizioSandri/RcppDeepState/pull/6)

Issues:
* [ Valgrind for initial pass](https://github.com/FabrizioSandri/RcppDeepState/issues/7)