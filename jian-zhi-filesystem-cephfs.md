# 建置 Filesystem CephFS

使用 CephFS 服務，得需要有個 matadata server，建立方式如下：

```bash
# ceph-deploy mds create {ceph-node}
ceph-deploy mds create node1
```

{% hint style="info" %}
此範例，是將 mds 服務建置在 mon-1 \ mgr 相同 node 上。  
參考連結：  
[http://docs.ceph.com/docs/master/start/quick-ceph-deploy/\#add-a-metadata-server](http://docs.ceph.com/docs/master/start/quick-ceph-deploy/#add-a-metadata-server)  
[http://docs.ceph.com/docs/mimic/rados/deployment/ceph-deploy-mds/](http://docs.ceph.com/docs/mimic/rados/deployment/ceph-deploy-mds/)
{% endhint %}



