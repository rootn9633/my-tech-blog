---
title: Setting up ZFS
date: "2019-12-16T22:12:03.284Z"
template: "post"
draft: true
slug: "/posts/Setting-up-ZFS/"
category: "ZFS"
tags:
  - "ZFS"
  - "zpool"
  - "freebsd"
description: "Setting up ZFS in freebsd"
---

## ZFS Pools

### creating a new zpool

```shell
zpool create [-fnd] [-o property=value] ...
           [-O file-system-property=value] ... [-m mountpoint] [-R root]
           [-t tempname] pool vdev ...
```

_Example:_

create a mirror (RAID 1) storage called “mypool” from two disks(\/dev\/ada1 \/dev\/ada2)

`zpool create mypool mirror /dev/ada1 /dev/ada2`

_Result:_

```shell
rootn-bsd01:~:% zpool status
  pool: mypool
 state: ONLINE
  scan: none requested
config:

NAME        STATE     READ WRITE CKSUM
mypool      ONLINE       0     0     0
  mirror-0  ONLINE       0     0     0
    ada1    ONLINE       0     0     0
    ada2    ONLINE       0     0     0

errors: No known data errors

  pool: zroot
 state: ONLINE
  scan: none requested
config:

NAME        STATE     READ WRITE CKSUM
zroot       ONLINE       0     0     0
  ada0p3    ONLINE       0     0     0

errors: No known data errors
```

```shell
% zpool list
NAME     SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
mypool  31.5G   316K  31.5G        -         -     0%     0%  1.00x  ONLINE  -
zroot   61.5G  2.32G  59.2G        -         -     2%     3%  1.00x  ONLINE  -
```

## ZFS Datasets

### Creating a new dataset

```shell
zfs create [-pu] [-o property=value]... filesystem
zfs create [-ps] [-b blocksize] [-o property=value]... -V size volume
```

_Example:_

Create a dataset *upload* under *mypool*
`% zfs create  mypool/upload`

_Result:_

```shell
% zfs list
NAME                 USED  AVAIL  REFER  MOUNTPOINT
mypool               488K  30.5G    88K  /home/ftp
mypool/upload         88K  30.5G    88K  /home/ftp/upload
```

### Mounting

```shell
zfs set property=value [property=value]... filesystem|volume|snapshot...
```

_Example:_
Mount *mypool* on *\/home\/ftp*
`% zfs set mountpoint=/home/ftp mypool`

_Result:_

```shell
% zfs list
NAME                 USED  AVAIL  REFER  MOUNTPOINT
mypool               488K  30.5G    88K  /mypool
...
```

### Setting other properties

```shell
zfs set property=value [property=value]... filesystem|volume|snapshot...
```

_Example:_

Setting *mypool* compression to *lz4*

`zfs set compress=lz4 mypool`

_Result:_

```shell
% zfs get all mypool
NAME    PROPERTY              VALUE                  SOURCE
mypool  type                  filesystem             -
...
mypool  compression           lz4                    local
...
```
