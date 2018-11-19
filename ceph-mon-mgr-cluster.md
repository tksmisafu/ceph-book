# Ceph mon / mgr Cluster

稍早，環境中已經建立第一個 mon / mgr 角色於 node1上，monitor 角色是不可中斷服務的一環  
故需要搭建第二、三台 monitor node，方式如下：

```
# ceph-deploy mon add {ceph-nodes}
ceph-deploy mon add node2 node3

# 當建置3個 mon node 之後，可查看仲裁狀況(quorum info)
ceph quorum_status --format json-pretty

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ #

# mgr role
ceph-deploy mgr create node2 node3

# check status
ssh node1 sudo ceph -s
```

搭建後架構如下

![](.gitbook/assets/image%20%285%29.png)

{% hint style="info" %}
參考連結：  
[http://docs.ceph.com/docs/master/start/quick-ceph-deploy/\#expanding-your-cluster](http://docs.ceph.com/docs/master/start/quick-ceph-deploy/#expanding-your-cluster)  
[http://docs.ceph.com/docs/master/start/quick-ceph-deploy/\#adding-managers](http://docs.ceph.com/docs/master/start/quick-ceph-deploy/#adding-managers)
{% endhint %}

