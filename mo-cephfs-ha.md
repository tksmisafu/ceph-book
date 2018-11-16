# 模擬 CephFS HA

### 模擬 mds fail

```bash
# ceph-mon-231 node
[ceph-admin@ceph-mon-231 prod-cluster]$ sudo ceph fs status
gitlab_fs - 1 clients
=========
+------+--------+--------------+---------------+-------+-------+
| Rank | State  |     MDS      |    Activity   |  dns  |  inos |
+------+--------+--------------+---------------+-------+-------+
|  0   | active | ceph-mon-231 | Reqs:    0 /s |   10  |   13  |
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
| ceph-mon-232 |
| ceph-mon-233 |
+--------------+
MDS version: ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)
[ceph-admin@ceph-mon-231 prod-cluster]$
[ceph-admin@ceph-mon-231 prod-cluster]$

################################################################################################
# 關閉 ceph-mds@ceph-mon-231 服務
[ceph-admin@ceph-mon-231 prod-cluster]$ sudo systemctl stop ceph-mds@ceph-mon-231
################################################################################################


# 下面呈現 CephFS rejoin 過程
[ceph-admin@ceph-mon-231 prod-cluster]$ sudo ceph fs status
gitlab_fs - 0 clients
=========
+------+--------+--------------+----------+-------+-------+
| Rank | State  |     MDS      | Activity |  dns  |  inos |
+------+--------+--------------+----------+-------+-------+
|  0   | rejoin | ceph-mon-233 |          |    0  |    0  |
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
| ceph-mon-232 |
+--------------+
MDS version: ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)

[ceph-admin@ceph-mon-231 prod-cluster]$ sudo ceph fs status
gitlab_fs - 1 clients
=========
+------+--------+--------------+---------------+-------+-------+
| Rank | State  |     MDS      |    Activity   |  dns  |  inos |
+------+--------+--------------+---------------+-------+-------+
|  0   | active | ceph-mon-233 | Reqs:    0 /s |   10  |   13  |
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
| ceph-mon-232 |
+--------------+
MDS version: ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)
[ceph-admin@ceph-mon-231 prod-cluster]$

################################################################################################
# mattermost-211 Ceph Client node
################################################################################################
[afu@mattermost-211 ~]$ sudo mount -l |grep ceph
10.1.13.231:6789:/ on /mnt/cephfs type ceph (rw,relatime,name=admin,secret=<hidden>,acl,wsize=16777216)
[afu@mattermost-211 ~]$ ss -anut |grep ESTAB |grep '10.1.13.231'
tcp    ESTAB      0      0      10.1.13.211:33980              10.1.13.231:6789
tcp    ESTAB      0      0      10.1.13.211:41850              10.1.13.231:6801
[afu@mattermost-211 ~]$
[afu@mattermost-211 ~]$
[afu@mattermost-211 ~]$ sudo mount -l |grep ceph
10.1.13.231:6789:/ on /mnt/cephfs type ceph (rw,relatime,name=admin,secret=<hidden>,acl,wsize=16777216)
# 以上是模擬前狀態
# 以下是模擬後狀態
[afu@mattermost-211 ~]$ ss -anut |grep ESTAB |grep '10.1.13.231'
tcp    ESTAB      0      0      10.1.13.211:33980              10.1.13.231:6789
[afu@mattermost-211 ~]$
[afu@mattermost-211 ~]$ ss -anut |grep ESTAB |grep '10.1.13.233'
tcp    ESTAB      0      0      10.1.13.211:48104              10.1.13.233:6800
[afu@mattermost-211 ~]$
```

### 模擬 Ceph Mon fail

```bash
################################################################################################
# 關閉 ceph-mon@ceph-mon-231 服務
################################################################################################

[ceph-admin@ceph-mon-231 prod-cluster]$ sudo systemctl stop ceph-mon@ceph-mon-231
[ceph-admin@ceph-mon-231 prod-cluster]$
[ceph-admin@ceph-mon-231 prod-cluster]$
[ceph-admin@ceph-mon-231 prod-cluster]$ sudo ceph fs status
gitlab_fs - 1 clients
=========
+------+--------+--------------+---------------+-------+-------+
| Rank | State  |     MDS      |    Activity   |  dns  |  inos |
+------+--------+--------------+---------------+-------+-------+
|  0   | active | ceph-mon-232 | Reqs:    0 /s |   23  |   14  |
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
| ceph-mon-233 |
+--------------+
MDS version: ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)
[ceph-admin@ceph-mon-231 prod-cluster]$

################################################################################################
# 觀察 mattermost-211 'Ceph Client note'
################################################################################################

[afu@mattermost-211 ~]$
[afu@mattermost-211 ~]$ sudo mount -l |grep ceph
10.1.13.231:6789:/ on /mnt/cephfs type ceph (rw,relatime,name=admin,secret=<hidden>,acl,wsize=16777216)
# 上述維持 Mount 狀態

[afu@mattermost-211 ~]$ ss -anut |grep ESTAB |grep '10.1.13.23'
tcp    ESTAB      0      0      10.1.13.211:33996              10.1.13.231:6789
tcp    ESTAB      0      0      10.1.13.211:42704              10.1.13.232:6800
[afu@mattermost-211 ~]$
# 上面呈現 Ceph Client 維持與 mon.1 連線
# 下面呈現 Ceph Client 變成與 mon.3 連線
[afu@mattermost-211 ~]$
[afu@mattermost-211 ~]$ ss -anut |grep ESTAB |grep '10.1.13.23'
tcp    ESTAB      0      0      10.1.13.211:52138              10.1.13.233:6789
tcp    ESTAB      0      0      10.1.13.211:42704              10.1.13.232:6800
[afu@mattermost-211 ~]$
[afu@mattermost-211 ~]$
```

