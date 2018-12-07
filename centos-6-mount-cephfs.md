# CentOS 6 掛載 CephFS

針對 CentOS 6 環境掛載 CephFS 需求，特寫此篇紀錄

### Ceph Server Mon 端必要動作

#### Ceph Mon 端針對 CentOS 6 Client 端指定舊版本進行安裝

```bash
# 先確認 crush tunables profile = hammer 設定
$ sudo ceph osd crush show-tunables |grep profile
    "profile": "hammer",
# 上述檢查後如不是，則下指令，並持續觀察 ceph status
$ ceph osd crush tunables hammer
 
$ ceph status
  cluster:
    id:     4ab0a924-xxxx-oooo-bbbb-4d70697ba106
    health: HEALTH_OK
# 最後針對個別用戶端進行套件安裝
# 192.168.1.226 = CentOS 6 Host
sudo ceph-deploy install root@192.168.1.226 --release hammer
```

### CentOS 6 Ceph 用戶端動作

```bash
# 先前紀錄
# yum install yum-plugin-priorities epel-release
# rpm -ivh https://download.ceph.com/rpm-hammer/el6/noarch/ceph-release-1-1.el6.noarch.rpm
# yum install ceph ceph-fuse ceph-common nfs-utils
# 上面三個動作，可透過（ sudo ceph-deploy install root@192.168.1.226 --release hammer）取代。
# ～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
# 以下是實務上可行做法。
sudo yum -y install ceph-fuse nfs-utils
sudo scp user@ceph-mon-ip:/etc/ceph/ceph.c* /etc/ceph/
sudo mkdir /mnt/cephfs/

# 進行 mount
sudo ceph-fuse -m ceph-mds-ip:6789,ceph-mds-ip2:6789 /mnt/cephfs/
sudo ceph-fuse -m ceph-mds-ip:6789,ceph-mds-ip2:6789 -r /subdir /mnt/cephfs/

# 設定開機後自動 mount，故意設定於 rc.local，千萬不要使用 fstab
sudo echo "ceph-fuse -m ceph-mds-ip:6789,ceph-mds-ip2:6789 -r /subdir /mnt/mycephfs/" >> /etc/rc.local

# 如有需要進行 umount
$ fusermount -u /mnt/mycephfs/
```

