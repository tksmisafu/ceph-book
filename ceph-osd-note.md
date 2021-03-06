# OSD 新增與管理

## 新增 OSD

```text
# 新增實體硬碟作為 OSD
ceph-deploy osd create --data /dev/sdb node1
ceph-deploy osd create --data /dev/sdb node2
ceph-deploy osd create --data /dev/sdb node3
```

另一種模擬方式：添加兩個 OSD，因模擬環境，先以目錄方式模擬 OSD～

```bash
# 產出模擬的 OSD
ssh node2
sudo mkdir /var/local/osd0
exit

ssh node3
sudo mkdir /var/local/osd1
exit

# 準備 OSD
ceph-deploy osd prepare node2:/var/local/osd0 node3:/var/local/osd1
# 啟用 OSD：“ ceph-deploy osd activate {ceph-node}:/path/to/directory ”
ceph-deploy osd activate node2:/var/local/osd0 node3:/var/local/osd1

# 檢查集群健康狀態
ceph health
```

{% hint style="info" %}
`osd prepare` + `osd activate` = `osd create`  
[http://docs.ceph.com/docs/master/ceph-volume/lvm/create/](http://docs.ceph.com/docs/master/ceph-volume/lvm/create/)
{% endhint %}

{% hint style="info" %}
peering 完成后，集群应该达到`active + clean`状态。  
OSD 目錄：/var/lib/ceph/osd/{osd.id}
{% endhint %}

{% hint style="info" %}
文章出處：  
[http://docs.ceph.org.cn/start/quick-ceph-deploy/\#id2](http://docs.ceph.org.cn/start/quick-ceph-deploy/#id2)
{% endhint %}

## 配置 OSD

```bash
[osd]
osd_journal = /tmp/ceph/journal
osd_journal_size = 5120
```

{% hint style="info" %}
[http://docs.ceph.com/docs/mimic/rados/configuration/osd-config-ref/](http://docs.ceph.com/docs/mimic/rados/configuration/osd-config-ref/)  
[http://docs.ceph.com/docs/mimic/rados/configuration/journal-ref/](http://docs.ceph.com/docs/mimic/rados/configuration/journal-ref/)
{% endhint %}

## 重啟/觀察 OSD 服務

```bash
# in OSD node, service start/stop/restart
sudo systemctl stop ceph-osd@*
sudo systemctl start ceph-osd@*
sudo systemctl restart ceph-osd@*

# monitor OSD
sudo systemctl status ceph-osd@*
sudo ceph -w
sudo ceph osd status
```

## OSD in / out service

```bash
sudo ceph osd out|in {osd.id}
sudo ceph osd in 1
sudo ceph osd out 1
```

## Remove OSD

```bash
# in OSD node，remove osd step
# step 1 先將 osd out
sudo ceph osd out 1
# step 2 停止 osd service
sudo systemctl stop ceph-osd@1
# step 3 purge all osd data from the monitors.
sudo ceph osd purge 1 --yes-i-really-mean-it
# step 4 remove lvm layer
sudo ceph-volume lvm zap /dev/sdc --destroy

##### 實作 #####
sudo ceph osd out 1
sudo systemctl stop ceph-osd@1
# osd purge <osdname (id|osd.id)> {--yes-i-really-mean-it}  
# purge all osd data from the monitors. Combines `osd destroy`, `osd rm`, and `osd crush rm`.
sudo ceph osd purge 1 --yes-i-really-mean-it
  >> purged osd.1

# monitor OSD status
sudo ceph -s
sudo ceph-volume lvm list

# ?
sudo ls -la /etc/systemd/system/multi-user.target.wants/ -1
sudo systemctl disable ceph-volume@lvm-1-ed22e4cd-adfc-4201-8b5f-732a44e8f236.service

#sudo umount /var/lib/ceph/osd/ceph-1

# 執行 ceph-volume lvm zap /dev/sdc --destroy 之前觀察
sudo lsblk -a
sudo fdisk -l
sudo ceph-volume lvm list

# 執行 ceph-volume lvm zap /dev/sdc --destroy
# 意思是移除此磁碟的相關 LV VG PV 關係
sudo ceph-volume lvm zap /dev/sdc --destroy
Running command: /usr/sbin/cryptsetup status /dev/mapper/
 stdout: /dev/mapper/ is inactive.
--> Zapping: /dev/sdc
--> Unmounting /var/lib/ceph/osd/ceph-1
Running command: /bin/umount -v /var/lib/ceph/osd/ceph-1
 stderr: umount: /var/lib/ceph/osd/ceph-1 (tmpfs) unmounted
--> Destroying volume group ceph-676d3fd7-888e-487f-8f1a-b76648a596ed because --destroy was given
Running command: /usr/sbin/vgremove -v -f ceph-676d3fd7-888e-487f-8f1a-b76648a596ed
 stderr: Removing ceph--676d3fd7--888e--487f--8f1a--b76648a596ed-osd--block--ed22e4cd--adfc--4201--8b5f--732a44e8f236 (253:3)
 stderr: Archiving volume group "ceph-676d3fd7-888e-487f-8f1a-b76648a596ed" metadata (seqno 17).
    Releasing logical volume "osd-block-ed22e4cd-adfc-4201-8b5f-732a44e8f236"
 stderr: Creating volume group backup "/etc/lvm/backup/ceph-676d3fd7-888e-487f-8f1a-b76648a596ed" (seqno 18).
 stdout: Logical volume "osd-block-ed22e4cd-adfc-4201-8b5f-732a44e8f236" successfully removed
 stderr: Removing physical volume "/dev/sdc" from volume group "ceph-676d3fd7-888e-487f-8f1a-b76648a596ed"
 stdout: Volume group "ceph-676d3fd7-888e-487f-8f1a-b76648a596ed" successfully removed
Running command: /usr/sbin/wipefs --all /dev/sdc
 stdout: /dev/sdc: 8 bytes were erased at offset 0x00000218 (LVM2_member): 4c 56 4d 32 20 30 30 31
Running command: /bin/dd if=/dev/zero of=/dev/sdc bs=1M count=10
 stderr: 10+0 records in
10+0 records out
10485760 bytes (10 MB) copied
 stderr: , 0.01328 s, 790 MB/s
--> Zapping successful for: /dev/sdc
```

{% hint style="info" %}
文章出處：  
[http://docs.ceph.com/docs/mimic/rados/operations/add-or-rm-osds/\#removing-osds-manual](http://docs.ceph.com/docs/mimic/rados/operations/add-or-rm-osds/#removing-osds-manual)  
[http://docs.ceph.com/docs/master/ceph-volume/lvm/zap/](http://docs.ceph.com/docs/master/ceph-volume/lvm/zap/)  
手動移除的步驟：  
[https://access.redhat.com/documentation/en-us/red\_hat\_ceph\_storage/2/html/administration\_guide/managing\_cluster\_size\#removing\_an\_osd](https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/2/html/administration_guide/managing_cluster_size#removing_an_osd)
{% endhint %}

## 監控 OSD

```bash
sudo ceph -w | -s
  -s, --status          show cluster status
  -w, --watch           watch live cluster changes

[afu@dev-ceph-osd1 ~]$ ceph osd tree
ID CLASS WEIGHT  TYPE NAME              STATUS REWEIGHT PRI-AFF
-1       0.10623 root default
-3       0.02829     host dev-ceph-osd1
 0   hdd 0.02829         osd.0              up  1.00000 1.00000
-5       0.03897     host dev-ceph-osd2
 2   hdd 0.01949         osd.2              up  1.00000 1.00000
 3   hdd 0.01949         osd.3              up  1.00000 1.00000
-7       0.03897     host dev-ceph-osd3
 4   hdd 0.01949         osd.4              up  1.00000 1.00000
 5   hdd 0.01949         osd.5              up  1.00000 1.00000

[afu@dev-ceph-osd1 ~]$ ceph osd status
+----+--------------------------------+-------+-------+--------+---------+--------+---------+-----------+
| id |              host              |  used | avail | wr ops | wr data | rd ops | rd data |   state   |
+----+--------------------------------+-------+-------+--------+---------+--------+---------+-----------+
| 0  | dev-ceph-osd1.kingbay-tech.com | 10.0G | 18.9G |    0   |     0   |    0   |     0   | exists,up |
| 2  | dev-ceph-osd2.kingbay-tech.com | 1077M | 18.9G |    0   |     0   |    0   |     0   | exists,up |
| 3  | dev-ceph-osd2.kingbay-tech.com | 1078M | 18.9G |    0   |     0   |    0   |     0   | exists,up |
| 4  | dev-ceph-osd3.kingbay-tech.com | 1078M | 18.9G |    0   |     0   |    0   |     0   | exists,up |
| 5  | dev-ceph-osd3.kingbay-tech.com | 1142M | 18.8G |    0   |     0   |    0   |     0   | exists,up |
+----+--------------------------------+-------+-------+--------+---------+--------+---------+-----------+


[afu@dev-ceph-osd1 ~]$ ceph osd stat
5 osds: 5 up, 5 in; epoch: e426


[afu@dev-ceph-osd2 ~]$ sudo ceph-volume lvm list
====== osd.3 =======

  [block]    /dev/ceph-67430e84-0df0-4548-834a-3cb703394694/osd-block-86e4258f-0251-4e04-ac6b-0231350c91bf

      type                      block
      osd id                    3
      cluster fsid              4333408c-d6d4-4bd5-a856-d3a295e7e793
      cluster name              ceph
      osd fsid                  86e4258f-0251-4e04-ac6b-0231350c91bf
      encrypted                 0
      cephx lockbox secret
      block uuid                yt9thV-U60a-7bne-cnK3-gy9x-pqoF-oxgV3s
      block device              /dev/ceph-67430e84-0df0-4548-834a-3cb703394694/osd-block-86e4258f-0251-4e04-ac6b-0231350c91bf
      vdo                       0
      crush device class        None
      devices                   /dev/sdc

[afu@host-206 ~]$ sudo ceph osd perf
osd commit_latency(ms) apply_latency(ms)
  8                  0                 0
  1                  0                 0
 15                  0                 0
 11                  0                 0
 10                  0                 0
 12                  0                 0
 16                  0                 0
 13                  0                 0
 19                 13                13
 18                  4                 4
  3                  0                 0
 14                  0                 0
  9                  0                 0
  7                  0                 0
  0                  0                 0
  4                  0                 0
  5                  0                 0
  6                  0                 0
 17                  0                 0
  2                  0                 0
```

[http://docs.ceph.com/docs/mimic/rados/configuration/mon-osd-interaction](http://docs.ceph.com/docs/mimic/rados/configuration/mon-osd-interaction)

## OSD Cache

Bluestore use bcache

