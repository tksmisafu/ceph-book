# 錯誤：rbd image uses unsupported features: 0x38

rbd 新增時未指定 --image-feature

```bash
[ceph-admin@ceph-mon-1 cluster]$ sudo rbd create --size 512000 p_rbd/gitlab_bk
# 使用上述 rbd 進行 map 時出現錯誤：rbd image uses unsupported features: 0x38
# 因為 CentOS 內核僅支持其中 feature layering，其他 feature 概不支持
# 所以 rbd create 必須有 --image-feature layering
[ceph-admin@ceph-mon-1 cluster]$ sudo rbd create --image-feature layering --size 512000 p_rbd/gitlab_bk

```

