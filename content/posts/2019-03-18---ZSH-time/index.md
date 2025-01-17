---
title: zsh-time
date: "2019-03-18T22:12:03.284Z"
template: "post"
draft: false
slug: "/posts/zsh-time/"
category: "Command Line"
tags:
  - "command line"
  - "zsh"
  - "time"
description: "how to display more details using the time command in zsh"
---

## Reason

I installed zsh in on my personal computer and servers, and uses the time command to view how much time a process took. However, once I wanted to see also how much memory a process allocated, and found that the time command could do much more.
The original time format:

```bash
TIMEFMT='%J  %U user %S system %P cpu %*E total'
```

To see more information, I changed it to:

```bash
TIMEFMT='%J   %U  user %S system %P cpu %*E total'$'\n'\
'avg shared (code):         %X KB'$'\n'\
'avg unshared (data/stack): %D KB'$'\n'\
'total (sum):               %K KB'$'\n'\
'max memory:                %M KB'$'\n'\
'page faults from disk:     %F'$'\n'\
'other page faults:         %R'
```
