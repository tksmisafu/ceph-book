# 壓測 CephFS vs Ceph RBD

在 Ceph-client host\(vm\) 端，準備好 20GB size file，同時 mount cephfs、ceph rbd 兩個服務

## 單一大檔案 測試

### CP file to CephFS

```bash
[afu@client-211 ~]$ time sudo cp /tmp/26G.tar /mnt/cephfs/

real    3m45.307s
user    0m0.024s
sys     0m32.702s

# 網路 iftop peak: 900Mb
```

![](.gitbook/assets/image%20%287%29.png)

### CP file to Ceph RBD

```bash
[afu@client-211 ~]$ time sudo cp /tmp/26G.tar /mnt/ceph-rbd/

real    3m44.799s
user    0m0.204s
sys     0m43.456s

# 網路 iftop peak: 901Mb
```

![](.gitbook/assets/image%20%284%29.png)

## 單一目錄零碎資料

8.4G /ios    241514 files  
3.6G /mobile

### CP file to CephFS

```bash
[afu@client-206 ~]$ time sudo cp -r /home/vic/dev/gitlab_migration/files/backup/git_backup/ios /mnt/cephfs/

real    11m30.740s
user    0m2.758s
sys     0m38.308s


[afu@backup-206 ~]$ time sudo cp -r /home/vic/dev/gitlab_migration/files/backup/git_backup/mobile /mnt/cephfs/

real    4m11.318s
user    0m1.224s
sys     0m15.183s
```

![](.gitbook/assets/image%20%283%29.png)

### CP file to Ceph RBD

```bash
[afu@client-206 ~]$ time sudo cp -r /home/vic/dev/gitlab_migration/files/backup/git_backup/ios /mnt/ceph-rbd/

real    6m52.891s
user    0m2.118s
sys     0m34.511s
# 網路 iftop peak: 719Mb

[afu@backup-206 ~]$ time sudo cp -r /home/vic/dev/gitlab_migration/files/backup/git_backup/mobile /mnt/ceph-rbd/

real    2m40.833s
user    0m0.866s
sys     0m14.679s
```

## 目錄探索 du 指令

```bash
[afu@backup-206 ~]$ time du -sh /mnt/cephfs/ios /mnt/cephfs/mobile
8.4G    /mnt/cephfs/ios
3.6G    /mnt/cephfs/mobile

real    2m19.288s
user    0m1.129s
sys     0m4.361s

[afu@backup-206 ~]$ time du -sh /mnt/ceph-rbd/ios /mnt/ceph-rbd/mobile
9.4G    /mnt/ceph-rbd/ios
3.9G    /mnt/ceph-rbd/mobile

real    0m38.699s
user    0m0.673s
sys     0m3.698s
```

目錄檔案數量 find 指令

```bash
[afu@backup-206 ~]$ time sudo find /mnt/ceph-rbd/ios/ -type f | wc -l
241514

real    0m1.691s    ~    0m1.484s
user    0m0.515s
sys     0m1.142s

[afu@backup-206 ~]$ time sudo find /mnt/cephfs/ios/ -type f | wc -l
241514

real    0m26.703s / 0m5.881s / 0m1.546s ~ 0m1.642s
user    0m0.658s
sys     0m1.643s

# 首次執行 find 指令，耗時久，第二~三次後時間逐漸縮短。
```

測試心得

```text
Ceph RBD & CephFS 複製單一大檔案，耗時相當
Ceph RBD & CephFS 複製單一目錄內涵24萬個檔案，RBD 耗時相當於 cephfs 的 60% 時間
Ceph RBD & CephFS 計算相同大小目錄的容量，RBD 耗時相當於 cephfs 的 28% 時間
Ceph RBD & CephFS 計算相同目錄的檔案數量，RBD 耗時相當於 cephfs 穩定。
```

