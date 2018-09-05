---
layout: post
title:  ceph install (mimic filestore)
date:   2018-09-04 15:20:58
categories: Ceph
tags: Ceph
---


# Ceph
how to install ceph mimic 13.2.1(stable)

## Softwere
linux version: CentOS Linux release 7.5.1804 (Core)  
ceph version: mimic 13.2.1(stable)   
ceph-deploy version: 2.0.0  

## Config
|    hostname    | ip1 |ip2|disk|
| ----------  | :---: |:---:|---|
|ceph01|10.10.111.21|10.10.115.100|sdb-h and sda4-10|
|ceph02|10.10.111.22|10.10.115.101|sdb-h and sda4-10|
|ceph03|10.10.111.23|10.10.115.102|sdb-h and sda4-10|
|ceph04|10.10.111.24|10.10.115.103|sdb-h and sda4-10|
|ceph05|10.10.111.25|10.10.115.104|sdb-h and sda4-10|
The partition format of each disk is GPT (operaing system are included)

## Generate ssh-key
~~~
#ssh-keygen
~~~
##
## Config hosts
Add the following content to /etc/hosts
~~~
10.10.111.21 ceph01
10.10.111.22 ceph02
10.10.111.23 ceph03
10.10.111.24 ceph04
10.10.111.25 ceph05
~~~
## copy pub key to ceph01-05
```bash
ssh-copy-id root@ceph01
ssh-copy-id root@ceph02
ssh-copy-id root@ceph03
ssh-copy-id root@ceph04
ssh-copy-id root@ceph05
```
## test
```shell
for i in ceph01 ceph02 ceph03 ceph04 ceph05
do
ssh $i
done
```
## install packages
~~~
yum install wget net-tools vim openssh-server ntp ntpdate ntp-doc
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
~~~
## yum repo
~~~
# cat /etc/yum.repos.d/ceph.repo
[Ceph]
name=Ceph packages for $basearch
baseurl=http://mirrors.163.com/ceph/rpm-luminous/el7/$basearch
enabled=1
priority=1
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[Ceph-noarch]
name=Ceph noarch packages
baseurl=http://mirrors.163.com/ceph/rpm-luminous/el7/noarch
enabled=1
priority=1
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[ceph-source]
name=Ceph source packages
baseurl=http://mirrors.163.com/ceph/rpm-luminous/el7/SRPMS
enabled=0
priority=1
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc
~~~

## install ceph-deploy
```shell
yum install ceph-deploy -y
cpeh-deploy --version
# version 2.0.0
```
## start to install ceph
~~~
ceph-deploy install ceph01 ceph02 ceph03 ceph04 ceph05
# Actually hosts it dones
# yum install epel-release
# then yum install install ceph ceph-radosgw ceph-common
~~~

## Config cluster
I decided to deploy three of mon and mgr
```shell
mkdir /my-cluster
cd /my-cluster
```
The next things you have to do it in this directory(my-cluster)
for example to clear cluster information, when you find or do something is wrong you can do this
~~~
ceph-deploy purge ceph01 ceph02 ceph03 ceph04 ceph05
ceph-deploy purgedata ceph01 ceph02 ceph03 ceph04 ceph05
ceph-deploy forgetkeys
rm ceph.*
~~~
then start again
~~~
ceph-deploy new ceph01
~~~
that would be Generate three of file
~~~
ceph.conf ceph.mon.keyring ceph-deploy-ceph.log
~~~
edit Config
~~~
[global]
fsid = 369b310d-4bcd-4b7b-9fc1-365e73ec5a34
mon_initial_members = ceph01 ceph02 ceph03
mon_host = 10.10.111.21,10.10.111.22,10.10.111.23
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
~~~

## initial monitor
~~~
ceph-deploy mon create-initial
~~~
it would initial there of ceph01 ceph02 ceph03
check proceses
~~~
ps -ef|grep cpeh
ceph      126540       1  0 Sep04 ?        00:02:29 /usr/bin/ceph-mon -f --cluster ceph --id ceph01 --setuser ceph --setgroup ceph
~~~
## config network  
edit config of ceph.conf
~~~
public network  = 10.10.111.0/24
cluster network = 10.10.115.0/24
~~~
push ceph.conf to ceph01-05
~~~
ceph-deploy --overwrite-conf config push ceph01-05
~~~

## create keph keyring
~~~
# ceph-deploy gatherkeys ceph01
# ll /my-cluster
-rw-------. 1 root root    113 Sep  4 17:01 ceph.bootstrap-mds.keyring
-rw-------. 1 root root    113 Sep  4 17:01 ceph.bootstrap-mgr.keyring
-rw-------. 1 root root    113 Sep  4 17:01 ceph.bootstrap-osd.keyring
-rw-------. 1 root root    113 Sep  4 17:01 ceph.bootstrap-rgw.keyring
-rw-------. 1 root root    151 Sep  4 17:01 ceph.client.admin.keyring
~~~
check the cluster health
~~~
ceph --keyring ceph.client.admin.keyring -c ceph.conf -s

cluster:
  id:     369b310d-4bcd-4b7b-9fc1-365e73ec5a34
  health: HEALTH_OK

services:
  mon: 3 daemons, quorum ceph01,ceph02
  mgr: 0
  osd: 0 osds: 0 up, 0 in
  rgw: 0 daemons active

data:
  pools:   0 pools, 0 pgs
  objects: 0  objects, 0 KiB
  usage:   0 GiB used, 0 TiB / 0 TiB avail
  pgs:     0 active+clean

# ceph --keyring ceph.client.admin.keyring -c ceph.conf auth get client.admin
~~~
push kerring to ceph01-05
~~~
ceph-deploy admin ceph01-05
then
ceph -s
~~~
## create mgr
~~~
ceph-deploy mgr create ceph01 ceph02 ceph03
ceph -s
cluster:
  id:     369b310d-4bcd-4b7b-9fc1-365e73ec5a34
  health: HEALTH_OK

services:
  mon: 3 daemons, quorum ceph01,ceph02
  mgr: ceph01(active), standbys: ceph03, ceph02
  osd: 0 osds: 0 up, 0 in
  rgw: 0 daemons active

data:
  pools:   0 pools, 0 pgs
  objects: 0  objects, 0 KiB
  usage:   0 GiB used, 0 TiB / 0 TiB avail
  pgs:     0 active+clean
~~~
## Add osds
I used is filestore not bluestore
~~~
for i in ceph01 ceph02 ceph03 ceph04 ceph05; do ceph-deploy osd create --filestore --data /dev/sdb --journal /dev/sda4 $i; ceph-deploy osd create --filestore --data /dev/sdc --journal /dev/sda5 $i; ceph-deploy osd create --filestore --data /dev/sdd --journal /dev/sda6 $i; ceph-deploy osd create --filestore --data /dev/sde --journal /dev/sda7 $i; ceph-deploy osd create --filestore --data /dev/sdf --journal /dev/sda8 $i; ceph-deploy osd create --filestore --data /dev/sdg --journal /dev/sda9 $i; ceph-deploy osd create --filestore --data /dev/sdh --journal /dev/sda10 $i; done
~~~
check mount
~~~
[root@ceph01 ceph-0]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  100G  2.5G   98G   3% /
devtmpfs                  16G     0   16G   0% /dev
tmpfs                     16G   16K   16G   1% /dev/shm
tmpfs                     16G   42M   16G   1% /run
tmpfs                     16G     0   16G   0% /sys/fs/cgroup
/dev/sda2                2.0G  172M  1.8G   9% /boot
tmpfs                    3.2G     0  3.2G   0% /run/user/0
/dev/dm-2                7.3T  118M  7.3T   1% /var/lib/ceph/osd/ceph-0
/dev/dm-3                7.3T  118M  7.3T   1% /var/lib/ceph/osd/ceph-1
/dev/dm-4                7.3T  118M  7.3T   1% /var/lib/ceph/osd/ceph-2
/dev/dm-5                7.3T  118M  7.3T   1% /var/lib/ceph/osd/ceph-3
/dev/dm-6                7.3T  118M  7.3T   1% /var/lib/ceph/osd/ceph-4
/dev/dm-7                7.3T  119M  7.3T   1% /var/lib/ceph/osd/ceph-5
/dev/dm-8                7.3T  118M  7.3T   1% /var/lib/ceph/osd/ceph-6
~~~
## content
~~~
ll  /var/lib/ceph/osd/ceph-0/
-rw-r--r--.   1 ceph ceph  402 Sep  4 09:18 activate.monmap
-rw-r--r--.   1 ceph ceph   37 Sep  4 09:18 ceph_fsid
drwxr-xr-x. 172 ceph ceph 8192 Sep  5 10:54 current
-rw-r--r--.   1 ceph ceph   37 Sep  4 09:18 fsid
lrwxrwxrwx.   1 root root    9 Sep  4 09:18 journal -> /dev/sda4
-rw-------.   1 ceph ceph   56 Sep  4 09:18 keyring
-rw-r--r--.   1 ceph ceph   21 Sep  4 09:18 magic
-rw-r--r--.   1 ceph ceph   41 Sep  4 09:18 osd_key
-rw-r--r--.   1 ceph ceph    6 Sep  4 09:18 ready
-rw-r--r--.   1 ceph ceph    4 Sep  4 09:18 store_version
-rw-r--r--.   1 ceph ceph   53 Sep  4 09:18 superblock
-rw-r--r--.   1 ceph ceph   10 Sep  4 09:18 type
-rw-r--r--.   1 ceph ceph    2 Sep  4 09:18 whoami
check status
~~~

## Add rgw
~~~
ceph-deploy install --rgw ceph04 ceph05
~~~
### Access url
~~~
curl ceph04:7480(default port)
~~~

~~~
[root@ceph01 my-cluster]# ceph -s
  cluster:
    id:     369b310d-4bcd-4b7b-9fc1-365e73ec5a34
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum ceph01,ceph02,localhost
    mgr: ceph01(active), standbys: ceph03, ceph02
    osd: 35 osds: 35 up, 35 in
    rgw: 2 daemons active

  data:
    pools:   5 pools, 1064 pgs
    objects: 220  objects, 1.1 KiB
    usage:   4.0 GiB used, 255 TiB / 255 TiB avail
    pgs:     1064 active+clean
~~~

## create pools
~~~
ceph osd pool create vm_disk 256 256
ceph osd pool create vm_image 256 256
ceph osd pool create rbd 256 256
ceph -s
-----------------
[root@ceph01 ceph-0]# ceph -s
  cluster:
    id:     369b310d-4bcd-4b7b-9fc1-365e73ec5a34
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum ceph01,ceph02,localhost
    mgr: ceph01(active), standbys: ceph03, ceph02
    osd: 35 osds: 35 up, 35 in
    rgw: 2 daemons active

  data:
    pools:   8 pools, 1064 pgs
    objects: 220  objects, 1.1 KiB
    usage:   4.0 GiB used, 255 TiB / 255 TiB avail
    pgs:     1064 active+clean

  io:
    client:   1.7 KiB/s rd, 0 B/s wr, 1 op/s rd, 1 op/s wr
~~~
