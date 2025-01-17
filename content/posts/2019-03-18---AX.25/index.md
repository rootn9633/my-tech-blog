---
title: AX.25
date: "2019-03-18T22:12:03.284Z"
template: "post"
draft: false
slug: "/posts/AX.25/"
category: "APRS"
tags:
  - "APRS"
  - "AX.25"
description: "Since APRS uses AX.25, I just want to explore it more in detail"
---

## spec

* OSI layer 2
* no defined physical layer
  * typically  on VHF(very high frequency), UHF(ultra high frequency)
* [HDLC](/HDLC) transmitted with [NRZI](/NRZI)
* has a variable length address field containing a source address, zero or more repeater addresses and a destination address.
* the addresses derive from the station call signs.
* Media access control follows CSMA/CR

## Reference

1. [Chepponis, M., & Karn, P. (1987, August). The KISS TNC: A simple Host-to-TNC communications protocol. In /6th Computer Networking Conference/ (Vol. 225, pp. 06111-1494).](http://www.ka9q.net/papers/kiss.html)
