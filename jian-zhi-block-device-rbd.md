# 建置 Block Device RBD​

### Add new RBD \(BLOCK DEVICE POOL\)

在管理節點上，先透過 ceph-deploy 將 Ceph 套件安装到目標主機上。  
再進行配置檔案傳送。

```bash
# install ceph packege in ceph-client Host。
ceph-deploy install {ceph-client}
# copy the "Ceph configuration file" and the "ceph.client.admin.keyring" to the ceph-client Host.
ceph-deploy admin {ceph-client}
```

新增、初始化 RBD 之前，須先確認`pool create`是否已經新增並且`enable {pool-name} rbd`  
確認後方可進行 RBD init。

```bash
# On the admin node, use the ceph tool to create a pool.
ceph osd pool create <pool-name> 128

# On the admin node, use the rbd tool to initialize the pool for use by RBD:
rbd pool init <pool-name>

# CREATING A BLOCK DEVICE IMAGE
rbd create --size {megabytes} {pool-name}/{image-name}
```

```bash
# 範例
[ceph-admin@ceph-mon-1 cluster]$ sudo ceph osd pool create p_rbd 128
pool 'p_rbd' created
[ceph-admin@ceph-mon-1 cluster]$ sudo rbd pool init p_rbd
[ceph-admin@ceph-mon-1 cluster]$

# 512000 = 500GB
[ceph-admin@ceph-mon-1 cluster]$ sudo rbd create --size 512000 p_rbd/gitlab_bk
[ceph-admin@ceph-mon-1 cluster]$
```

### 查看 RBD 資訊

```bash
[ceph-admin@ceph-mon-1 cluster]$ sudo rbd ls p_rbd
gitlab_bk

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
```

### CREATE A BLOCK DEVICE USER

```bash
# ceph auth get-or-create client.gitrbd mon 'profile rbd' osd 'profile rbd pool=p_rbd'
[ceph-admin@ceph-mon-1 cluster]$ sudo ceph auth get-or-create client.gitrbd mon 'profile rbd' osd 'profile rbd pool=p_rbd'
[client.gitrbd]
        key = AQAFM/Jb57X0JRAAKfNJOAsRXkqMByhHXSPW8w==
```

### Ceph-Client Host ~ CONFIGURE A BLOCK DEVICE

```bash
# On the ceph-client node, create a block device image.
# rbd create [image-name] --size 4096 --image-feature layering [-m {mon-IP}] [-k /path/to/ceph.client.admin.keyring]

# On the ceph-client node, map the image to a block device.
sudo rbd map [image-name] --name client.admin [-m {mon-IP}] [-k /path/to/ceph.client.admin.keyring]

# Use the block device by creating a file system on the ceph-client node.
sudo mkfs.ext4 -m0 /dev/rbd/rbd/[image-name]
    > This may take a few moments.

# Mount the file system on the ceph-client node.
sudo mkdir /mnt/ceph-block-device
sudo mount /dev/rbd/rbd/foo /mnt/ceph-block-device
cd /mnt/ceph-block-device

#
```

```bash
# 範例
[afu@client-211 ~]$ sudo rbd map p_rbd/gitlab_bk --name client.admin -m 192.168.13.231 -k /etc/ceph/ceph.client.admin.keyring
/dev/rbd0

[afu@client-211 ~]$ dmesg | tail
[242204.141703] libceph: mon0 192.168.13.231:6789 session established
[242204.142084] libceph: client74148 fsid 4ab0a924-e5fd-4048-8cef-4d70697ba106
[242204.149233] rbd: rbd0: capacity 536870912000 features 0x1
```

{% hint style="info" %}
參考文件：  
[http://docs.ceph.com/docs/mimic/start/quick-rbd/](http://docs.ceph.com/docs/mimic/start/quick-rbd/)
{% endhint %}

### automatically mapped and mounted at boot 

{% hint style="info" %}
參考文件：  
[http://docs.ceph.com/docs/mimic/man/8/rbdmap/](http://docs.ceph.com/docs/mimic/man/8/rbdmap/)
{% endhint %}

### Add RBD Cache

`rbd cache = true` to the `[client]` section of your `ceph.conf` file.   
更細項設定，參考下列文件。

{% hint style="info" %}
參考文件：  
[http://docs.ceph.com/docs/mimic/rbd/rbd-config-ref/](http://docs.ceph.com/docs/mimic/rbd/rbd-config-ref/)
{% endhint %}

