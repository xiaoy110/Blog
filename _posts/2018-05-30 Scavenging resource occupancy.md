---
layout: post
title:  ceph resize disk
date:   2018-02-27 15:20:58
categories: Ceph
tags: Ceph
---


# ceph
~~~
[root@node1 ~]# rbd status runner-git
Watchers:
	watcher=10.10.107.104:0/3620166064 client.9136635 cookie=4
	watcher=10.10.107.103:0/1399062367 client.9084497 cookie=11
	watcher=10.10.107.105:0/1068730776 client.9084492 cookie=16

canot delete it.

[root@node1 mnt]# rbd rm runner-git
2017-03-13 14:21:35.958280 7fce9702b7c0 -1 librbd: image has watchers - not removing
Removing image: 0% complete...failed.
rbd: error: image still has watchers
This means the image is still open or the client using it crashed. Try again after closing/unmapping it or waiting 30s for the crashed client to timeout
通过查看ceph info 来找到blockname

[root@node1 mnt]# rbd info runner-git| grep prefix
        block_name_prefix: rbd_data.1031238e1f29
使用listwatcher定位到哪些ip在使用

[root@node1 mnt]# rados -p rbd listwatchers rbd_header.8b9e8a238e1f29
watcher=10.10.107.104:0/3620166064 client.9136635 cookie=4
watcher=10.10.107.103:0/1399062367 client.9084497 cookie=11
watcher=10.10.107.105:0/1068730776 client.9084492 cookie=16
只要定位到那个机器再使用就比较好解决了。

ssh 10.117.130.149
[root@client ~]# rbd showmapped
id pool image snap device    
0  rbd  foo   -    /dev/rbd0
1  rbd  foo   -    /dev/rbd1
[root@client ~]# rbd unmap /dev/rbd0
[root@client ~]# rbd unmap /dev/rbd1
[root@client ~]# rbd showmapped
~~~
