---
layout: post
title:  ceph resize disk
date:   2018-02-37 15:20:58
categories: Ceph
tags: Ceph
---


# ceph
## 查看ceph有多少个pool
ceph osd lspools
ceph osd pool ls

## 检查 pool 大小
rados df
ceph df

## pool里面有多少镜像
rbd ls pool_name
 
## 删除镜像
delete image
rbd snap purge vm_image/20151014_10-52-10.img
rbd rm -p vm_image 20151014_10-52-10.img

## 增大
rbd resize --size 20480 foo 

## 缩小
rbd resize --size 150000 tm-img --allow-shrink

## 此时还需要做如下
resize2fs /dev/rbd0

## 获取大小
blockdev --getsize64 /dev/rbd0

