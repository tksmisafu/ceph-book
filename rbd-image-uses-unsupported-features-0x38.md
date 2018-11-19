# 問題：rbd image uses unsupported features: 0x38

rbd 新增時未指定 --image-feature

```bash
[ceph-admin@ceph-mon-1 cluster]$ sudo rbd create --size 512000 p_rbd/gitlab_bk
# 使用上述 rbd 進行 map 時出現錯誤：rbd image uses unsupported features: 0x38
# 因為 CentOS 內核僅支持其中 feature layering，其他 feature 概不支持
# 所以 rbd create 必須有 --image-feature layering
[ceph-admin@ceph-mon-1 cluster]$ sudo rbd create --image-feature layering --size 512000 p_rbd/gitlab_bk

```

有無 --image-feature 之差異觀察

```bash
[ceph-admin@ceph-mon-1 cluster]$ sudo rbd info p_rbd/gitlab_bk
rbd image 'gitlab_bk':
        size 500 GiB in 128000 objects
        order 22 (4 MiB objects)
        id: fcf66b8b4567
        block_name_prefix: rbd_data.fcf66b8b4567
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        op_features:
        flags:
        create_timestamp: Mon Nov 19 11:23:50 2018

##########################################################################################

[ceph-admin@ceph-mon-1 cluster]$ sudo rbd create --image-feature layering --size 512000 p_rbd/gitlab_bk
[ceph-admin@ceph-mon-1 cluster]$ sudo rbd info p_rbd/gitlab_bk
rbd image 'gitlab_bk':
        size 500 GiB in 128000 objects
        order 22 (4 MiB objects)
        id: 121956b8b4567
        block_name_prefix: rbd_data.121956b8b4567
        format: 2
        features: layering
        op_features:
        flags:
        create_timestamp: Mon Nov 19 12:25:15 2018

```

