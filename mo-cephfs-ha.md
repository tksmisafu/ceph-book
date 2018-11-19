# 模擬 CephFS HA

### 模擬 mds fail

```bash
# ceph-mon-1 node
[ceph-admin@ceph-mon-1 cluster]$ sudo ceph fs status
gitlab_fs - 1 clients
=========
+------+--------+--------------+---------------+-------+-------+
| Rank | State  |     MDS      |    Activity   |  dns  |  inos |
+------+--------+--------------+---------------+-------+-------+
|  0   | active | ceph-mon-1   | Reqs:    0 /s |   10  |   13  |
+------+--------+--------------+---------------+-------+-------+
+--------------------+----------+-------+-------+
|        Pool        |   type   |  used | avail |
+--------------------+----------+-------+-------+
| gitlab_fs_metadata | metadata | 2943  | 46.0T |
|   gitlab_fs_data   |   data   |    0  | 46.0T |
+--------------------+----------+-------+-------+
+--------------+
| Standby MDS  |
+--------------+
| ceph-mon-2   |
| ceph-mon-3   |
+--------------+
MDS version: ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)
[ceph-admin@ceph-mon-1 cluster]$
[ceph-admin@ceph-mon-1 cluster]$

################################################################################################
# 關閉 ceph-mds@ceph-mon-1 服務
[ceph-admin@ceph-mon-1 cluster]$ sudo systemctl stop ceph-mds@ceph-mon-1
################################################################################################


# 下面呈現 CephFS rejoin 過程
[ceph-admin@ceph-mon-1 cluster]$ sudo ceph fs status
gitlab_fs - 0 clients
=========
+------+--------+--------------+----------+-------+-------+
| Rank | State  |     MDS      | Activity |  dns  |  inos |
+------+--------+--------------+----------+-------+-------+
|  0   | rejoin | ceph-mon-3   |          |    0  |    0  |
+------+--------+--------------+----------+-------+-------+
+--------------------+----------+-------+-------+
|        Pool        |   type   |  used | avail |
+--------------------+----------+-------+-------+
| gitlab_fs_metadata | metadata | 2943  | 46.0T |
|   gitlab_fs_data   |   data   |    0  | 46.0T |
+--------------------+----------+-------+-------+
+--------------+
| Standby MDS  |
+--------------+
| ceph-mon-2   |
+--------------+
MDS version: ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)

[ceph-admin@ceph-mon-1 cluster]$ sudo ceph fs status
gitlab_fs - 1 clients
=========
+------+--------+--------------+---------------+-------+-------+
| Rank | State  |     MDS      |    Activity   |  dns  |  inos |
+------+--------+--------------+---------------+-------+-------+
|  0   | active | ceph-mon-3   | Reqs:    0 /s |   10  |   13  |
+------+--------+--------------+---------------+-------+-------+
+--------------------+----------+-------+-------+
|        Pool        |   type   |  used | avail |
+--------------------+----------+-------+-------+
| gitlab_fs_metadata | metadata | 3059  | 46.0T |
|   gitlab_fs_data   |   data   |    0  | 46.0T |
+--------------------+----------+-------+-------+
+--------------+
| Standby MDS  |
+--------------+
| ceph-mon-2   |
+--------------+
MDS version: ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)
[ceph-admin@ceph-mon-1 cluster]$

################################################################################################
# client-211 Ceph Client node
################################################################################################
[afu@client-211 ~]$ sudo mount -l |grep ceph
192.168.13.231:6789:/ on /mnt/cephfs type ceph (rw,relatime,name=admin,secret=<hidden>,acl,wsize=16777216)
[afu@client-211 ~]$ ss -anut |grep ESTAB |grep '192.168.13.231'
tcp    ESTAB      0      0      192.168.13.211:33980              192.168.13.231:6789
tcp    ESTAB      0      0      192.168.13.211:41850              192.168.13.231:6801
[afu@client-211 ~]$
[afu@client-211 ~]$
[afu@client-211 ~]$ sudo mount -l |grep ceph
192.168.13.231:6789:/ on /mnt/cephfs type ceph (rw,relatime,name=admin,secret=<hidden>,acl,wsize=16777216)
# 以上是模擬前狀態
# 以下是模擬後狀態
[afu@client-211 ~]$ ss -anut |grep ESTAB |grep '192.168.13.231'
tcp    ESTAB      0      0      192.168.13.211:33980              192.168.13.231:6789
[afu@client-211 ~]$
[afu@client-211 ~]$ ss -anut |grep ESTAB |grep '192.168.13.233'
tcp    ESTAB      0      0      192.168.13.211:48104              192.168.13.233:6800
[afu@client-211 ~]$
```

### 模擬 Ceph Mon fail

```bash
################################################################################################
# 關閉 ceph-mon@ceph-mon-1 服務
################################################################################################

[ceph-admin@ceph-mon-1 cluster]$ sudo systemctl stop ceph-mon@ceph-mon-1
[ceph-admin@ceph-mon-1 cluster]$
[ceph-admin@ceph-mon-1 cluster]$
[ceph-admin@ceph-mon-1 cluster]$ sudo ceph fs status
gitlab_fs - 1 clients
=========
+------+--------+--------------+---------------+-------+-------+
| Rank | State  |     MDS      |    Activity   |  dns  |  inos |
+------+--------+--------------+---------------+-------+-------+
|  0   | active | ceph-mon-2   | Reqs:    0 /s |   23  |   14  |
+------+--------+--------------+---------------+-------+-------+
+--------------------+----------+-------+-------+
|        Pool        |   type   |  used | avail |
+--------------------+----------+-------+-------+
| gitlab_fs_metadata | metadata |  108k | 46.0T |
|   gitlab_fs_data   |   data   |   14  | 46.0T |
+--------------------+----------+-------+-------+
+--------------+
| Standby MDS  |
+--------------+
| ceph-mon-3   |
+--------------+
MDS version: ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)
[ceph-admin@ceph-mon-1 cluster]$

################################################################################################
# 觀察 client-211 'Ceph Client note'
################################################################################################

[afu@client-211 ~]$
[afu@client-211 ~]$ sudo mount -l |grep ceph
192.168.13.231:6789:/ on /mnt/cephfs type ceph (rw,relatime,name=admin,secret=<hidden>,acl,wsize=16777216)
# 上述維持 Mount 狀態

[afu@client-211 ~]$ ss -anut |grep ESTAB |grep '192.168.13.23'
tcp    ESTAB      0      0      192.168.13.211:33996              192.168.13.231:6789
tcp    ESTAB      0      0      192.168.13.211:42704              192.168.13.232:6800
[afu@client-211 ~]$
# 上面呈現 Ceph Client 維持與 mon.1 連線
# 下面呈現 Ceph Client 變成與 mon.3 連線
[afu@client-211 ~]$
[afu@client-211 ~]$ ss -anut |grep ESTAB |grep '192.168.13.23'
tcp    ESTAB      0      0      192.168.13.211:52138              192.168.13.233:6789
tcp    ESTAB      0      0      192.168.13.211:42704              192.168.13.232:6800
[afu@client-211 ~]$
[afu@client-211 ~]$
```

### 模擬 mds master change

```bash
[afu@ceph-mon-2 ~]$ sudo ceph fs status
gitlab_fs - 1 clients
=========
+------+--------+--------------+---------------+-------+-------+
| Rank | State  |     MDS      |    Activity   |  dns  |  inos |
+------+--------+--------------+---------------+-------+-------+
|  0   | active | ceph-mon-3   | Reqs:    0 /s |  164  |  150  |
+------+--------+--------------+---------------+-------+-------+
+--------------------+----------+-------+-------+
|        Pool        |   type   |  used | avail |
+--------------------+----------+-------+-------+
| gitlab_fs_metadata | metadata |  885k | 46.0T |
|   gitlab_fs_data   |   data   |  294k | 46.0T |
+--------------------+----------+-------+-------+
+--------------+
| Standby MDS  |
+--------------+
| ceph-mon-1   |
| ceph-mon-2   |
+--------------+
MDS version: ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)

################################################################################################
# disable MDS master node 'ceph-mon-3'
[afu@ceph-mon-2 ~]$ sudo ceph mds fail ceph-mon-3
failed mds gid 44495
################################################################################################

[afu@ceph-mon-2 ~]$ sudo ceph fs status
gitlab_fs - 1 clients
=========
+------+--------+--------------+---------------+-------+-------+
| Rank | State  |     MDS      |    Activity   |  dns  |  inos |
+------+--------+--------------+---------------+-------+-------+
|  0   | active | ceph-mon-2   | Reqs:    0 /s |  164  |  150  |
+------+--------+--------------+---------------+-------+-------+
+--------------------+----------+-------+-------+
|        Pool        |   type   |  used | avail |
+--------------------+----------+-------+-------+
| gitlab_fs_metadata | metadata |  885k | 46.0T |
|   gitlab_fs_data   |   data   |  294k | 46.0T |
+--------------------+----------+-------+-------+
+--------------+
| Standby MDS  |
+--------------+
| ceph-mon-1   |
| ceph-mon-3   |
+--------------+
MDS version: ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)
```

### 模擬恢復 mds & mon.1 尚未恢復

```bash
# Ceph Client node 尚未 mount 狀態
[afu@client-211 ~]$ sudo mount -t ceph 192.168.13.231:6789:/ /mnt/cephfs -o name=admin,secretfile=/etc/ceph/admin.secret
mount error 110 = Connection timed out

# 成功 mount
[afu@client-211 ~]$ sudo mount -t ceph 192.168.13.232:6789:/ /mnt/cephfs -o name=admin,secretfile=/etc/ceph/admin.secret

[afu@client-211 ~]$ sudo mount -l |grep ceph
192.168.13.232:6789:/ on /mnt/cephfs type ceph (rw,relatime,name=admin,secret=<hidden>,acl,wsize=16777216)

[afu@client-211 ~]$ ss -anut |grep ESTAB |grep '192.168.13.23'
tcp    ESTAB      0      0      192.168.13.211:55092              192.168.13.232:6800
tcp    ESTAB      0      0      192.168.13.211:41934              192.168.13.232:6789
[afu@client-211 ~]$

```

{% hint style="info" %}
心得：  
Monitor & MDS 角色皆有 3 node 維持運作，有不錯的 HA基礎，如果再透過 virtual ip & keepalived 此機制，對 HA 機制更友善。
{% endhint %}

