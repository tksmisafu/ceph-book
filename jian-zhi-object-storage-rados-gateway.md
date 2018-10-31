# 建置 Object storage RADOS Gateway​

使用 Object 儲存服務，得需要有個 RGW instance，建立方式如下：

```
# ceph-deploy rgw create {gateway-node}
ceph-deploy rgw create node1

# RGW instance will listen on port 7480
# can be changed by editing ceph.conf 
[client]
rgw frontends = civetweb port=80
```

{% hint style="info" %}
參考連結：  
[http://docs.ceph.com/docs/master/start/quick-ceph-deploy/\#add-an-rgw-instance](http://docs.ceph.com/docs/master/start/quick-ceph-deploy/#add-an-rgw-instance)
{% endhint %}



