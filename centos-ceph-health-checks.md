# CentOS 7 Ceph health-checks

### 集群狀態查看

```bash
# 檢查集群健康狀態，下面兩個方式皆可
ceph health
ssh node1 sudo ceph health

# 查看集群 cluster\services\data 狀態，下面兩個方式皆可
ceph status
ssh node1 sudo ceph -s

```

### Cluster & Pool size 查看

```bash
# full ratio 查看
[ceph-admin@dev-ceph-mon ceph-cluster]$ ceph osd dump | grep full_ratio
full_ratio 0.95
backfillfull_ratio 0.9
nearfull_ratio 0.85

# Size 查看
[ceph-admin@dev-ceph-mon ceph-cluster]$ ceph df
GLOBAL:
    SIZE        AVAIL      RAW USED     %RAW USED
    109 GiB     95 GiB       14 GiB         13.09
POOLS:
    NAME                    ID     USED        %USED     MAX AVAIL     OBJECTS
    .rgw.root               1      1.1 KiB         0        22 GiB           4
    default.rgw.control     2          0 B         0        22 GiB           8
    default.rgw.meta        3          0 B         0        22 GiB           0
    default.rgw.log         4          0 B         0        22 GiB         207
```

{% hint style="info" %}
文章出處：  
[http://docs.ceph.com/docs/mimic/rados/operations/health-checks/](http://docs.ceph.com/docs/mimic/rados/operations/health-checks/)
{% endhint %}

### 集群 service 操作

基於 CentOS Redhat 系統環境，適用 sysvinit 運行 Ceph~

```bash
# 啟用 Ceph-daemon 
# sudo /etc/init.d/ceph [options] [start|restart] [daemonType|daemonID]
sudo /etc/init.d/ceph -a start    # [-a] is all node

# 停止 Ceph-daemon
# sudo /etc/init.d/ceph [options] stop [daemonType|daemonID]
sudo /etc/init.d/ceph -a stop

# 啟用指定的 daemon，例如 OSD
# sudo /etc/init.d/ceph [-a] start {daemon-type}
sudo /etc/init.d/ceph start osd
sudo /etc/init.d/ceph -a start osd

# 停止指定的 daemon，例如 OSD
# sudo /etc/init.d/ceph [-a] stop {daemon-type}
sudo /etc/init.d/ceph stop osd
sudo /etc/init.d/ceph -a stop osd

# 啟用｜停止 指定單一進程，例如 OSD
# sudo /etc/init.d/ceph [-a] start|stop {daemon-type}.{instance}
sudo /etc/init.d/ceph start osd.0
sudo /etc/init.d/ceph -a start osd.0

sudo /etc/init.d/ceph stop osd.0
sudo /etc/init.d/ceph -a stop osd.0
#####~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# 透過 service 進行管理，啟用｜停止 Ceph 服務
# sudo service ceph [options] [start|restart|stop] [daemonType|daemonID]
sudo service ceph -a start
sudo service ceph -a stop

# 透過 service 進行管理，啟用｜停止 指定的 Ceph-daemon 服務，例如 OSD
# sudo service ceph [-a] start|stop {daemon-type}
sudo service ceph -a start osd
sudo service ceph -a stop osd

# 透過 service 進行管理，啟用｜停止指定單一進程，例如 OSD
# sudo service ceph [-a] start|stop {daemon-type}.{instance}
sudo service ceph -a start osd.0
sudo service ceph -a stop osd.0

# 檢查集群狀態
sudo ceph health

```

{% hint style="info" %}
文章出處：  
[http://docs.ceph.org.cn/start/quick-ceph-deploy/\#id3](http://docs.ceph.org.cn/start/quick-ceph-deploy/#id3)  
[http://docs.ceph.org.cn/rados/operations/operating/\#sysvinit-ceph](http://docs.ceph.org.cn/rados/operations/operating/#sysvinit-ceph)
{% endhint %}

### 擴展集群

{% hint style="info" %}
文章出處：  
[http://docs.ceph.org.cn/start/quick-ceph-deploy/\#id4](http://docs.ceph.org.cn/start/quick-ceph-deploy/#id4)
{% endhint %}

