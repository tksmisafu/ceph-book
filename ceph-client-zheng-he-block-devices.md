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

