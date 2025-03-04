---
title: HDLC
date: "2019-03-18T22:12:03.284Z"
template: "post"
draft: false
slug: "/posts/HDLC/"
category: "APRS"
tags:
  - "APRS"
  - "HDLC"
description: "High-Level Data Link Control"
---

HDLC frames can be transmitted over synchronous or asynchronous serial communication links, thus the beginning and end of each frame has to be identified by a flag: 01111110(0x7E). It must be ensured that the flag doesn’t appear as data inside the frame.

## Synchronous

Seeing that the flag has six 1s in a row,  the other data is encoded such that it never contains more than five 1s consecutively. This is done by [Bit stuffing](https://en.wikipedia.org/wiki/Bit_stuffing): any time 5 consecutive 1s appear in the transmitted data, the data is paused and a 0 is transmitted.
Knowing of this method, every time it receives 5 consecutive 1s, the receiving device would strip out the following 0. If the sixth bit is 1, it is
 `"a flag" if seventh_bit==0 else "an error"`
When idle, a frame delimiter will be continuously transmitted. The HDLC spec allows the 0 bit at the end of the frame delimiter to be shared with the next one, thus 011111101111110…… . Note that some hardware doesn’t support this.
![image of waveform](./NrziEncodedFlags.png)
HDLC transmits with LSB first.

## Asynchronous

I’ll skip this part since APRS transmits synchronously.

## Reference

1. [High-Level Data Link Control - Wikipedia](https://en.wikipedia.org/wiki/High-Level_Data_Link_Control)
