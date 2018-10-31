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
ceph osd lspools
```

### SHOW POOL STATISTICS

```bash
rados df
```

### SET POOL QUOTAS



### DELETE A POOL



### RENAME A POOL



### MAKE A SNAPSHOT OF A POOL



### REMOVE A SNAPSHOT OF A POOL



### SET POOL VALUES



### GET POOL VALUES

{% hint style="info" %}
參考文件：  
[http://docs.ceph.com/docs/mimic/rados/operations/pools/\#create-a-pool](http://docs.ceph.com/docs/mimic/rados/operations/pools/#create-a-pool)​
{% endhint %}

