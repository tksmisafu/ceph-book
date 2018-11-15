# 問題：ceph health

```bash
[ceph-admin@ceph-mon-231 prod-cluster]$ sudo ceph health detail
HEALTH_WARN Reduced data availability: 600 pgs inactive
PG_AVAILABILITY Reduced data availability: 600 pgs inactive
    pg 8.112 is stuck inactive for 5030.042492, current state unknown, last acting []
    pg 8.113 is stuck inactive for 5030.042492, current state unknown, last acting []
    pg 8.114 is stuck inactive for 5030.042492, current state unknown, last acting []
    pg 8.115 is stuck inactive for 5030.042492, current state unknown, last acting []
    pg 8.116 is stuck inactive for 5030.042492, current state unknown, last acting []
    pg 8.117 is stuck inactive for 5030.042492, current state unknown, last acting []
    pg 8.118 is stuck inactive for 5030.042492, current state unknown, last acting []

```

```bash
[ceph-admin@ceph-mon-231 prod-cluster]$ sudo ceph -s
  cluster:
    id:     4ab0a924-e5fd-4048-8cef-4d70697ba106
    health: HEALTH_WARN
            Reduced data availability: 600 pgs inactive

  services:
    mon: 3 daemons, quorum ceph-mon-231,ceph-mon-232,ceph-mon-233
    mgr: ceph-mon-231(active), standbys: ceph-mon-232, ceph-mon-233
    mds: gitlab_fs-1/1/1 up  {0=ceph-mon-233=up:active}, 2 up:standby
    osd: 20 osds: 20 up, 20 in

  data:
    pools:   2 pools, 600 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     100.000% pgs unknown
             600 unknown

```

```bash
[ceph-admin@ceph-mon-231 prod-cluster]$ sudo ceph pg stat
600 pgs: 600 unknown; 0 B data, 0 B used, 0 B / 0 B avail

```

問題原因是：Monitor 沒有同步 MDS OSD 狀態  
解法：關閉防火牆 or 設定正確的防火牆策略

