# OSD BlueStore

### BlueStore OSD Create

```bash
# 啟用 WAL & DB，則需獨立 SSD GPT Partition(各別區分）
# 新建立 GPT Partition 在 SSD 磁碟上。
sudo fdisk /dev/sdc
Command (m for help): g
Building a new GPT disklabel (GUID: 244DE7B4-55D4-4D11-8B48-990B7977D1BC)


Command (m for help): n
Partition number (1-128, default 1):
First sector (2048-41943006, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-41943006, default 41943006):
Created partition 1


Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# 確認磁區 /dev/sdc1 取得 PARTUUID
[afu@dev-ceph-osd1 ~]$ sudo blkid
/dev/mapper/centos_c7--1804-root: UUID="c4db2fc5-7a90-46a7-8948-09e2f3b1cb42" TYPE="xfs"
/dev/sda2: UUID="f4pYhx-y0Gq-r1Vb-uVD2-fSIT-95zN-UgjyHW" TYPE="LVM2_member"
/dev/sda1: UUID="bd85f222-6315-4cde-b162-58f8f6f037a1" TYPE="xfs"
/dev/mapper/centos_c7--1804-swap: UUID="b982e808-832e-494d-8f3c-2b5fe76b958f" TYPE="swap"
/dev/sdc1: PTTYPE="dos" PARTUUID="de371b93-029b-448f-957b-a57b0f501858"

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# in Ceph admin node 新增 Bluestore OSD with WAL & DB
ceph-deploy osd create --data /dev/sdb --bluestore --block-db /dev/sdc1 --block-wal /dev/sdc2 dev-ceph-osd1
ceph-deploy osd create --data /dev/sdb --bluestore --block-db /dev/sdc1 --block-wal /dev/sdc2 dev-ceph-osd1
參數說明
    --data /dev/sdb          << OSD 磁碟區
    --block-db /dev/sdc1     << bluestore db 磁區
    --block-wal /dev/sdc2    << bluestore wal 磁區
    --bluestore              << bluestore OSD type
    dev-ceph-osd1            << OSD node

# 另一個方式
# in Ceph OSD node 新增 Bluestore OSD with WAL & DB
ceph-volume lvm create --data /dev/sdb --bluestore --block-db /dev/sdc1 --block-wal /dev/sdc2 dev-ceph-osd1
```

### 查看 OSD

```bash
# in Ceph OSD node
ceph-volume lvm list

# OSD 目錄
sudo ls -al /var/lib/ceph/osd/ceph-0
```

OSD  
--block.wal  
--block.db  
  
bluestore cache  
  
開啟壓縮

其中多個OSD可以共用相同的WAL和DB。  
[https://blog.skywebster.com/learning-ceph-part4-bluestore/](https://blog.skywebster.com/learning-ceph-part4-bluestore/)

[http://docs.ceph.com/docs/master/rados/configuration/bluestore-config-ref/](http://docs.ceph.com/docs/master/rados/configuration/bluestore-config-ref/)

