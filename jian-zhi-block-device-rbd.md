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
# On the admin node, use the rbd tool to initialize the pool for use by RBD:
rbd pool init <pool-name>
```

### Ceph-Client Host ~ CONFIGURE A BLOCK DEVICE

```bash
# On the ceph-client node, create a block device image.
rbd create [image-name] --size 4096 --image-feature layering [-m {mon-IP}] [-k /path/to/ceph.client.admin.keyring]

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

