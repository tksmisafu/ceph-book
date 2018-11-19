# Ceph client 整合測試 Block Devices

經過初次建立 RBD 並掛載使用後，進行一項測試

測試一

```text
# A host 掛載 rbd-C，並存放資料在裡頭


# A host 卸除 rbd-C


# 改由 B host 掛載 rbd-C，並存取原本裡頭的資料，是可以的。
```

測試二

```text
# Host map rbd 可否 read-only mode
```

兩台 Ceph-client 同時 map same rbd，同時 cp file 

```bash
[afu@client-211 ~]$ time sudo cp /tmp/26G.tar /mnt/ceph-rbd/26G-1.tar

real    3m44.475s
user    0m0.227s
sys     0m40.796s
# 網路 peak: 900Mb

[afu@client-206 ~]$ time sudo cp /tmp/10G.tar /mnt/ceph-rbd/

real    1m21.211s
user    0m0.086s
sys     0m15.643s
# 網路 peak: 908Mb
# 上述是同時進行複製，網路 peak 值也是同時，並且穩定。
```



