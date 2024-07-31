---
title: "[Agon Light] Agdev: Calling Assembly Code from C: A Practical Guide"
date: 2024-07-30T15:45:08+02:00
draft: true
toc: false
images:
tags: 
  - untagged
---

Integrating assembly code into a C program can be beneficial for optimizing performance-critical sections of your code, or when writing IO handling parts of code. Here’s a step-by-step guide on how to achieve this, using an example from the [GitHub repository](https://github.com/mikolajmikolajczyk/agdev-examples/tree/master/02-callasmfromc).

## Why combine Assembly in C?

AgDev is awesome in letting us use C/C++ for writing apps for Agon - but sometimes we need to dive deeper into assembly, for example:

* GPIO handling code - I couldn't find any easy way using C
* Time critical parts of code

## Setup

[Here](https://github.com/mikolajmikolajczyk/agdev-examples/tree/master/02-callasmfromc) is the repository if you are here only for snippets.

You will need:

1. AgDev installed and in PATH
2. Text editor
3. [agon-agdev-template](https://github.com/mikolajmikolajczyk/agon-agdev-template) cloned

## Introduction

In this blog post we will create small `math` library containing two functions:

* `add(a,b)`
* `sub(a,b)`

Our `math` library will be written in `assembly` and we will call this library from `main` function in C.

## Step-by-Step Guide

### Header file

First let's think how functions will look like. I came up with following definitions:

```C
uint_16_t add(int8_t a, uint8_t b);
uint_8_t add(int8_t a, uint8_t b);
```

Arguments and return
