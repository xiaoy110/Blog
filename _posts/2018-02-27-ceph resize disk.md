---
layout: post
title:  ceph resize disk
date:   2018-02-27 15:20:58
categories: Ceph
tags: Ceph
---


# ceph
1. 查看ceph有多少个pool
ceph osd lspools
ceph osd pool ls

2. 检查 pool 大小
rados df
ceph df

3. pool里面有多少镜像
rbd ls pool_name
 
4. 删除镜像
delete image
rbd snap purge vm_image/20151014_10-52-10.img
rbd rm -p vm_image 20151014_10-52-10.img

5. 增大
rbd resize --size 20480 foo 

6. 缩小
rbd resize --size 150000 tm-img --allow-shrink

7. 此时还需要做如下
resize2fs /dev/rbd0

8. 获取大小
blockdev --getsize64 /dev/rbd0

