---
title: random vs urandom in linux
date: "2021-07-18T00:00:00.000Z"
template: "post"
draft: false
slug: "/posts/linux-random-vs-urandom"
category: "linux"
tags:
  - "random"
  - "linux"
description: "what is the difference between /dev/random and /dev/urandom?"
---

> TL;DR: /dev/random is for top secret things, /dev/urandom is for all other normal things.

Whether it is for making a game or generating a key, plenty of programs needs access to random numbers. To fill this need, linux has two files `/dev/random` and `/dev/urandom` which provides an interface to the kernel's random number generator.

The way the random number generator(RNG) creates random numbers is by gathering environmental noise (hard drive rotation speed, intervals between keystrokes, etc.) and putting these randomness into an *entropy pool.* Random numbers can then be generated from this pool of randomness. The RNG also keeps and estimate of the number of bits of noise in the entropy pool.

## /dev/random vs /dev/urandom

`/dev/random` will only return random bytes within the estimated bits of noise, and blocks when the pool is empty until additional noise is gathered, thus it provides high quality random numbers. On the other hand, `/dev/urandom` will return regardless of the available randomness, and is theoretically vulnerable to attacks

> If you are unsure about whether you should use /dev/random or /dev/urandom, then probably you want to use the latter. As a general rule, /dev/urandom should be used for everything except long-lived GPG/SSL/SSH keys.

## Available randomness

As previously mentioned, the RNG keeps track of the available randomness in the entropy pool. This number can be read from the file `/proc/sys/kernel/random/entropy_avail`. Normally, this will be 4096 (bits), a full entropy pool.

## Reference

* [https://linux.die.net/man/4/random](https://linux.die.net/man/4/random)
* [https://serverfault.com/questions/172337/explain-in-plain-english-about-entropy-available](https://serverfault.com/questions/172337/explain-in-plain-english-about-entropy-available)