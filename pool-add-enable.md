# Pool add / enable / status

### Create Pool & Enable Pool

```bash
# CREATE A POOL
ceph osd pool create {pool-name} {pg-num} [{pgp-num}] [replicated] [crush-rule-name] [expected-num-objects]
ceph osd pool create {pool-name} {pg-num} {pgp-num} erasure [erasure-code-profile] [crush-rule-name] [expected_num_objects]

# ASSOCIATE POOL TO APPLICATION
# CephFS uses the application name cephfs, 
# RBD uses the application name rbd, and 
# RGW uses the application name rgw.
ceph osd pool application enable {pool-name} {application-name}
```

### LIST POOLS

```bash
[ceph-admin@dev-ceph-mon ceph-cluster]$ ceph osd lspools
1 .rgw.root
2 default.rgw.control
3 default.rgw.meta
4 default.rgw.log
```

### SHOW POOL STATISTICS

```bash
[ceph-admin@dev-ceph-mon ceph-cluster]$ rados df
POOL_NAME              USED OBJECTS CLONES COPIES MISSING_ON_PRIMARY UNFOUND DEGRADED RD_OPS      RD WR_OPS    WR
.rgw.root           1.1 KiB       4      0     12                  0       0        0     12   8 KiB      4 4 KiB
default.rgw.control     0 B       8      0     24                  0       0        0      0     0 B      0   0 B
default.rgw.log         0 B     207      0    621                  0       0        0 445891 435 MiB 297136   0 B
default.rgw.meta        0 B       0      0      0                  0       0        0      0     0 B      0   0 B

total_objects    219
total_used       14 GiB
total_avail      95 GiB
total_space      109 GiB
```

### SET POOL QUOTAS

```bash
ceph osd pool set-quota {pool-name} [max_objects {obj-count}] [max_bytes {bytes}]

# example:
ceph osd pool set-quota data max_objects 10000
```

### DELETE A POOL



### RENAME A POOL



### MAKE A SNAPSHOT OF A POOL



### REMOVE A SNAPSHOT OF A POOL



### SET POOL VALUES



### GET POOL VALUES

```bash
# osd pool get {pool-name} {key}
[ceph-admin@dev-ceph-mon ceph-cluster]$ ceph osd pool get .rgw.root size
size: 3
[ceph-admin@dev-ceph-mon ceph-cluster]$ ceph osd pool get .rgw.root min_size
min_size: 2
```

```bash
# demo ceph pool add && status
sudo ceph osd pool create afu-pool 0 0

sudo ceph osd pool get afu-pool size
size: 3

sudo ceph osd pool get afu-pool pg_num
pg_num: 1024

[ceph-admin@ceph-mon-231 prod-cluster]$ sudo ceph status
  cluster:
    id:     4ab0a924-e5fd-4048-8cef-4d70697ba106
    health: HEALTH_WARN
            Reduced data availability: 1024 pgs inactive

  services:
    mon: 3 daemons, quorum ceph-mon-231,ceph-mon-232,ceph-mon-233
    mgr: ceph-mon-231(active), standbys: ceph-mon-232, ceph-mon-233
    osd: 20 osds: 20 up, 20 in

  data:
    pools:   1 pools, 1024 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     100.000% pgs unknown
             1024 unknown
```

{% hint style="info" %}
參考文件：  
[http://docs.ceph.com/docs/mimic/rados/operations/pools/\#create-a-pool](http://docs.ceph.com/docs/mimic/rados/operations/pools/#create-a-pool)​
{% endhint %}



