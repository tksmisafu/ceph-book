# CentOS 7 Ceph health-checks

### 集群狀態查看

```bash
# 檢查集群健康狀態，下面兩個方式皆可
# ceph health

[ceph-admin@dev-ceph-mon ceph-cluster]$ ceph health
HEALTH_WARN too few PGs per OSD (19 < min 30)

[ceph-admin@dev-ceph-mon ceph-cluster]$ ceph health detail
HEALTH_WARN too few PGs per OSD (19 < min 30)
TOO_FEW_PGS too few PGs per OSD (19 < min 30)

# 查看集群 cluster\services\data 狀態，下面兩個方式皆可
[ceph-admin@dev-ceph-mon ceph-cluster]$ ceph status
  cluster:
    id:     ba84d7fd-92dd-4cca-a4c4-175a70c5e2ed
    health: HEALTH_WARN
            too few PGs per OSD (19 < min 30)

  services:
    mon: 1 daemons, quorum dev-ceph-mon
    mgr: dev-ceph-mon.kingbay-tech.com(active)
    osd: 5 osds: 5 up, 5 in
    rgw: 1 daemon active

  data:
    pools:   4 pools, 32 pgs
    objects: 219  objects, 1.1 KiB
    usage:   14 GiB used, 95 GiB / 109 GiB avail
    pgs:     32 active+clean
#
ssh node1 sudo ceph -s

```

```bash
# 查看 ceph-mon service status
[ceph-admin@ceph-mon-1 prod-cluster]$ systemctl status ceph-mon@ceph-mon-1
● ceph-mon@ceph-mon-1.service - Ceph cluster monitor daemon
   Loaded: loaded (/usr/lib/systemd/system/ceph-mon@.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2018-11-09 18:00:27 CST; 1min 10s ago
 Main PID: 9935 (ceph-mon)
   CGroup: /system.slice/system-ceph\x2dmon.slice/ceph-mon@ceph-mon-1.service
           └─9935 /usr/bin/ceph-mon -f --cluster ceph --id ceph-mon-1 --setuser ceph --setgroup ceph

# 查看 ceph-mgr service status
[ceph-admin@ceph-mon-1 prod-cluster]$ systemctl status ceph-mgr@ceph-mon-1
● ceph-mgr@ceph-mon-1.service - Ceph cluster manager daemon
   Loaded: loaded (/usr/lib/systemd/system/ceph-mgr@.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2018-11-09 18:00:27 CST; 1min 22s ago
 Main PID: 9921 (ceph-mgr)
   CGroup: /system.slice/system-ceph\x2dmgr.slice/ceph-mgr@ceph-mon-1.service
           └─9921 /usr/bin/ceph-mgr -f --cluster ceph --id ceph-mon-1 --setuser ceph --setgroup ceph
```

### Cluster & Pool size 查看

```bash
# full ratio 查看
[ceph-admin@dev-ceph-mon ceph-cluster]$ ceph osd dump | grep full_ratio
full_ratio 0.95
backfillfull_ratio 0.9
nearfull_ratio 0.85

# This is an early warning that the cluster is approaching full.
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

# One or more pools has reached its quota and is no longer allowing writes.
[ceph-admin@dev-ceph-mon ceph-cluster]$ ceph df detail
GLOBAL:
    SIZE        AVAIL      RAW USED     %RAW USED     OBJECTS
    109 GiB     95 GiB       14 GiB         13.09        219
POOLS:
    NAME                    ID     QUOTA OBJECTS     QUOTA BYTES     USED        %USED     MAX AVAIL     OBJECTS     DIRTY     READ        WRITE       RAW USED
    .rgw.root               1      N/A               N/A             1.1 KiB         0        22 GiB           4        4         12 B         4 B      3.4 KiB
    default.rgw.control     2      N/A               N/A                 0 B         0        22 GiB           8        8          0 B         0 B          0 B
    default.rgw.meta        3      N/A               N/A                 0 B         0        22 GiB           0        0          0 B         0 B          0 B
    default.rgw.log         4      N/A               N/A                 0 B         0        22 GiB         207      207      436 KiB     291 KiB          0 B
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

