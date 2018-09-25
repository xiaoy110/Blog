---
layout: post
title:  ceph operations
date:   2018-09-23 15:20:58
categories: Ceph
tags: Ceph
---


## Pool operations
~~~
1. 查看ceph有多少个pool
ceph osd lspools
ceph osd pool ls

2. 检查 pool 大小
rados df
ceph df

3. pool里面有多少镜像
rbd ls pool_name

4. create pool
ceph osd pool create <pool_name> 256(pg_size) 256(pgp_size)

5. delete pool
ceph osd pool delete <pool_name>  --yes-i-really-really-mean-it
~~~
## Image operations
~~~
1. Create a image
rbd create -image-format 2 <pool_name/image_name>

2. Check images
rbd ls <pool_name>
rbd info <pool_name/image_name>

3. Remove images
rbd rm <pool_name/image_name>

4. Copy images
rbd cp <pool_name/image_name>

5. Resize images
rbd resize <image_name> --size 50(MB)

6. Import images
import <srcpath> <destpool_name/destimage_name>

7. Export Images
import <srcpool_name/image_name> destimage_name

8. Clone disk
rbd clone <poolname/imagename@snapshotname> <newpoolname/imagename>

9. Create snaps
rbd snap create <poolname/imagename@snapshotname>

10. Protect snaps
rbd snap protect <poolname/imagename@snapshotname>

11. Remove protect snaps
rbd snap unprotect <poolname/imagename@snapshotname>

12. Remove snaps
rbd snap rm <poolname/imagename@snapshotname>

13. rbd snap rollback
rbd snap rollback <poolname/imagename@snapshotname>

14. Purge all snaps
rbd snap purge <poolname/imagename>

15. formation translate
qemu-img convert -f vpc tedt.vhd -O raw rbd:<VM_name>/disk1


~~~
