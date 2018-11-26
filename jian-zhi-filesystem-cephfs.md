# 建置 Filesystem CephFS

## Ceph server 準備工作

使用 CephFS 服務，得需要有個 matadata server，建立方式如下：

```bash
# ceph-deploy mds create {ceph-node}
ceph-deploy mds create node1
```

{% hint style="info" %}
此範例，是將 mds 服務建置在 mon-1 \ mgr 相同 node 上。  
參考連結：  
[http://docs.ceph.com/docs/master/start/quick-ceph-deploy/\#add-a-metadata-server](http://docs.ceph.com/docs/master/start/quick-ceph-deploy/#add-a-metadata-server)  
[http://docs.ceph.com/docs/mimic/rados/deployment/ceph-deploy-mds/](http://docs.ceph.com/docs/mimic/rados/deployment/ceph-deploy-mds/)
{% endhint %}

### CREATING POOLS

A Ceph filesystem requires at least two RADOS pools～  
one for data     and     one for metadata.

> * Using a higher replication level for the metadata pool, as any data loss in this pool can render the whole filesystem inaccessible.
> * Using lower-latency storage such as SSDs for the metadata pool, as this will directly affect the observed latency of filesystem operations on clients.

```bash
$ ceph osd pool create cephfs_data <pg_num>
$ ceph osd pool create cephfs_metadata <pg_num>

[ceph-admin@ceph-mon-1 cluster]$ sudo ceph osd pool create gitlab_fs_data 128
pool 'gitlab_fs_data' created
[ceph-admin@ceph-mon-1 cluster]$ sudo ceph osd pool create gitlab_fs_metadata 128
pool 'gitlab_fs_metadata' created
```

### enable the filesystem

```bash
# Once the pools are created, 
# you may enable the filesystem using the 'fs' new command:

$ ceph fs new <fs_name> <metadata> <data>
# 範例
[ceph-admin@ceph-mon-1 cluster]$ sudo ceph fs new gitlab_fs gitlab_fs_metadata gitlab_fs_data
new fs with metadata pool 11 and data pool 10
$ ceph fs new cephfs cephfs_metadata cephfs_data
$ ceph fs ls

# 查看 MDS 狀態
$ ceph mds stat
gitlab_fs-1/1/1 up  {0=ceph-mon-1=up:active}   << 這是範例
```

### 觀察 filesystem

```bash
[ceph-admin@ceph-mon-1 cluster]$ sudo ceph fs ls
name: gitlab_fs, metadata pool: p_fs_metadata, data pools: [p_fs_data ]

[ceph-admin@ceph-mon-1 cluster]$ sudo ceph mds stat
gitlab_fs-1/1/1 up  {0=ceph-mon-3=up:active}, 2 up:standby

[ceph-admin@ceph-mon-1 cluster]$ sudo ceph fs status
gitlab_fs - 0 clients
=========
+------+--------+--------------+---------------+-------+-------+
| Rank | State  |     MDS      |    Activity   |  dns  |  inos |
+------+--------+--------------+---------------+-------+-------+
|  0   | active | ceph-mon-3   | Reqs:    0 /s |    0  |    0  |
+------+--------+--------------+---------------+-------+-------+
+---------------+----------+-------+-------+
|      Pool     |   type   |  used | avail |
+---------------+----------+-------+-------+
| p_fs_metadata | metadata |    0  |    0  |
|   p_fs_data   |   data   |    0  |    0  |
+---------------+----------+-------+-------+
+--------------+
| Standby MDS  |
+--------------+
| ceph-mon-1   |
| ceph-mon-2   |
+--------------+
MDS version: ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)

```

{% hint style="warning" %}
問題：如何決定此 cephfs 空間大小？  
[http://docs.ceph.com/docs/mimic/cephfs/quota/](http://docs.ceph.com/docs/mimic/cephfs/quota/)
{% endhint %}

{% hint style="info" %}
參考連結：  
[http://docs.ceph.com/docs/master/cephfs/createfs/](http://docs.ceph.com/docs/master/cephfs/createfs/)
{% endhint %}

## MOUNT CEPHFS WITH THE KERNEL DRIVER

### To mount the Ceph file system

```bash
sudo mkdir /mnt/mycephfs
sudo yum install nfs-utils
sudo mount -t ceph 192.168.0.1:6789:/ /mnt/mycephfs
sudo mount -t ceph 192.168.100.170:6789:/ /mnt/mycephfs -o name=admin,secret=AQBXTdBbwQTHMBAAAmjSrBbh+ilLeXT1tpAyoA==

# 觀察
[afu@dev-jmeter01 ~]$ sudo dmesg | tail
[931972.858029] libceph: mon0 192.168.100.170:6789 session established
[931972.859088] libceph: client74238 fsid ba84d7fd-92dd-4cca-a4c4-175a70c5e2ed
```

> If you have more than one filesystem，  
> specify which one to mount using the `mds_namespace` option，  
> e.g. `-o mds_namespace=myfs`.

### To unmount the Ceph file system

```bash
sudo umount /mnt/mycephfs
```

{% hint style="info" %}
參考連結：  
[http://docs.ceph.com/docs/master/cephfs/kernel/](http://docs.ceph.com/docs/master/cephfs/kernel/)
{% endhint %}

## MOUNT CEPHFS USING FUSE

另有 ceph-fuse 工具指令，在 Ceph Client 端進行 mount file system～  
透過此步驟，需要於 Ceph Client 端完成複製兩個檔案，權限`chmod 644`  
`/etc/ceph/ceph.conf  
/etc/ceph/ceph.keyring`  
\(來自於 Ceph admin 節點上\)

### To mount the Ceph file system as a FUSE

```bash
# 範例
sudo mkdir /home/username/cephfs
sudo ceph-fuse -m 192.168.0.1:6789 /home/username/cephfs
```

### To automate mounting ceph-fuse

```text
# 範例 An example ceph-fuse mount on /mnt would
sudo systemctl start ceph-fuse@/mnt.service
sudo systemctl enable ceph-fuse@/mnt.service
```

> If you have more than one filesystem, specify which one to mount using the `--client_mds_namespace` command line argument，  
> or add a `client_mds_namespace`setting to your `ceph.conf`.

{% hint style="info" %}
參考連結：  
[http://docs.ceph.com/docs/master/cephfs/fuse/](http://docs.ceph.com/docs/master/cephfs/fuse/)  
[http://docs.ceph.com/docs/master/man/8/ceph-fuse/](http://docs.ceph.com/docs/master/man/8/ceph-fuse/)
{% endhint %}

## 開機啟動 mount 

```bash
[afu@client-211 ~]$ sudo vi /etc/fstab
192.168.13.232:6789:/    /mnt/cephfs    ceph    name=admin,secretfile=/etc/ceph/admin.secret,noatime,_netdev    0 2
```

{% hint style="info" %}
參考連結：  
[http://docs.ceph.com/docs/mimic/cephfs/fstab/](http://docs.ceph.com/docs/mimic/cephfs/fstab/)
{% endhint %}

## 移除 FS

```bash
[afu@ceph-mon-1 ~]$ sudo systemctl stop ceph-mds\*
[afu@ceph-mon-1 ~]$ sudo ceph mds fail 0
[afu@ceph-mon-1 ~]$ sudo ceph fs status
gitlab_fs - 0 clients
=========
+------+--------+-----+----------+-----+------+
| Rank | State  | MDS | Activity | dns | inos |
+------+--------+-----+----------+-----+------+
|  0   | failed |     |          |     |      |
+------+--------+-----+----------+-----+------+
+---------------+----------+-------+-------+
|      Pool     |   type   |  used | avail |
+---------------+----------+-------+-------+
| p_fs_metadata | metadata | 2286  | 46.0T |
|   p_fs_data   |   data   |    0  | 46.0T |
+---------------+----------+-------+-------+
+-------------+
| Standby MDS |
+-------------+
+-------------+
+---------+---------+
| version | daemons |
+---------+---------+
+---------+---------+
[afu@ceph-mon-1 ~]$ sudo ceph fs rm gitlab_fs --yes-i-really-mean-it
[afu@ceph-mon-1 ~]$
[afu@ceph-mon-1 ~]$ sudo ceph fs status
+-------------+
| Standby MDS |
+-------------+
+-------------+
+---------+---------+
| version | daemons |
+---------+---------+
+---------+---------+

[afu@ceph-mon-1 ~]$ sudo ceph osd pool delete p_fs_metadata --yes-i-really-really-mean-it
Error EPERM: WARNING: this will *PERMANENTLY DESTROY* all data stored in pool p_fs_metadata.  
If you are *ABSOLUTELY CERTAIN* that is what you want, pass the pool name *twice*, followed by --yes-i-really-really-mean-it.

[afu@ceph-mon-1 ~]$ sudo ceph osd pool delete p_fs_metadata p_fs_metadata --yes-i-really-really-mean-it
pool 'p_fs_metadata' removed
[afu@ceph-mon-1 ~]$
[afu@ceph-mon-1 ~]$ sudo ceph osd pool delete p_fs_data p_fs_data --yes-i-really-really-mean-it
pool 'p_fs_data' removed
[afu@ceph-mon-1 ~]$
[afu@ceph-mon-1 ~]$ sudo rm -rf /var/lib/ceph/mds/ceph-ceph-mon-1/
[afu@ceph-mon-1 ~]$
```

