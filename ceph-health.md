# 問題：mount fail

```bash
[afu@mattermost-211 root]$ sudo mount -t ceph 10.1.13.231:6789:/ /mnt/cephfs -o name=admin,secret=AQBQOeVbsoiwJhAAPOJ6BC8KWAymEhugQjRRsw==
mount: 10.1.13.231:6789:/: can't read superblock
[afu@mattermost-211 root]$
[afu@mattermost-211 root]$
[afu@mattermost-211 root]$ sudo dmesg | tail
[496482.476731] hpet1: lost 7 rtc interrupts
[497082.478467] hpet1: lost 7 rtc interrupts
[497682.483773] hpet1: lost 7 rtc interrupts
[498282.489305] hpet1: lost 7 rtc interrupts
[498834.545298] libceph: mon0 10.1.13.231:6789 session established
[498834.545994] libceph: client25029 fsid 4ab0a924-e5fd-4048-8cef-4d70697ba106
[498882.484344] hpet1: lost 5 rtc interrupts
[499014.324099] nf_conntrack version 0.5.0 (65536 buckets, 262144 max)
[499019.343734] ip_tables: (C) 2000-2006 Netfilter Core Team
[499019.368400] nf_conntrack version 0.5.0 (65536 buckets, 262144 max)

```

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

