# 壓測

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

![](.gitbook/assets/image%20%286%29.png)

### CP file to Ceph RBD

```bash
[afu@client-211 ~]$ time sudo cp /tmp/26G.tar /mnt/ceph-rbd/

real    3m44.799s
user    0m0.204s
sys     0m43.456s

# 網路 iftop peak: 901Mb
```

![](.gitbook/assets/image%20%284%29.png)

## 單一目錄零碎資料 10G

### CP file to CephFS

![](.gitbook/assets/image%20%283%29.png)

### CP file to Ceph RBD

