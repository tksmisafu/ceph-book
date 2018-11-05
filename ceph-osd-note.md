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


sudo ceph osd out 1
sudo systemctl stop ceph-osd@1
# osd purge <osdname (id|osd.id)> {--yes-i-really-mean-it}  
# purge all osd data from the monitors. Combines `osd destroy`, `osd rm`, and `osd crush rm`.
sudo ceph osd purge 1 --yes-i-really-mean-it
  >> purged osd.1

# monitor OSD status
sudo ceph -s
sudo ceph-volume lvm list

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

sudo ceph osd tree
```

[http://docs.ceph.com/docs/mimic/rados/configuration/mon-osd-interaction](http://docs.ceph.com/docs/mimic/rados/configuration/mon-osd-interaction)

## OSD Cache

Bluestore use bcache

