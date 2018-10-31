# 用CentOS 7環境建置 Ceph

我用 VMware 環境，去模擬、建置Ceph環境

建立三台 OSD node  每台 4Core \ 4GB RAM \ 60GB\|20GB\|20GB HDD \ CentOS7  
一台 Monitor node  4Core \ 4GB RAM \ 60GB HDD \ CentOS7

{% hint style="info" %}
作業系統的選擇考量：  
[http://docs.ceph.com/docs/master/start/os-recommendations/](http://docs.ceph.com/docs/master/start/os-recommendations/)
{% endhint %}

### 部署節點前，環境準備

#### 安装 CEPH 部署工具與環境

SELinux 设置为 Permissive or Disable && 確認系統校時

```text
# selinux set Permissive (或者也可關閉)
sudo setenforce 0
```

```bash
# 先不考慮啟用 yum plugin
# sudo yum install yum-plugin-priorities
```

安裝 ceph-deploy 

```bash
sudo yum install -y yum-utils && sudo yum-config-manager --add-repo https://dl.fedoraproject.org/pub/epel/7/x86_64/ && sudo yum install --nogpgcheck -y epel-release && sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7 && sudo rm /etc/yum.repos.d/dl.fedoraproject.org*
sudo vim /etc/yum.repos.d/ceph.repo
#
[ceph-noarch]
name=Ceph noarch packages
baseurl=http://download.ceph.com/rpm-mimic/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
#
sudo yum update && sudo yum install ceph-deploy
```

#### 創建 “部署 CEPH” 的用户 && 允许免密碼 SSH

> ceph-deploy 工具必須以普通用户連線 Ceph 节点，且此用户可免密码使用 sudo 的權限，因為它需要在安装套件及配置文件的過程中，不必输入密碼。  
> 在 CentOS 系統上，要確保此帳號關閉了 requiretty 設置，避免 ceph-deploy 執行上出錯。

```bash
ssh user@ceph-server
sudo useradd -d /home/{username} -m {username}
sudo passwd {username}

echo "{username} ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/{username}
echo "Defaults:{username} !requiretty" | sudo tee /etc/sudoers.d/{username}
sudo chmod 0440 /etc/sudoers.d/{username}
su {username}
ssh-keygen
ssh-copy-id {username}@node1
ssh-copy-id {username}@node2
ssh-copy-id {username}@node3
```

#### 開設網路存取權 && 設定 Firewalld

```bash
sudo firewall-cmd --zone=public --add-port=6789/tcp --permanent
```

### 創建集群

基於管理方便，先在管理節點上創建一個目錄，用於保存 ceph-deploy 所產出的配置文件與密鑰。並且在後續執行 ceph-deploy 時，是在此新目錄內下指令。

```bash
mkdir my-cluster
cd my-cluster
# 初始創建集群，在 Monitor 節點執行指令（initial-monitor-node）
ceph-deploy new node1
# 上述執行後，會產出三份檔案：ceph.conf 配置文件、ceph.mon.keyring 密鑰和 ceph-deploy-ceph.log 日誌文件。

# 如模擬兩個OSD環境下，設定副本數可以呈現 “active + clean” 系統狀態
# ceph.conf 文件內 [global] 段落
osd pool default size = 2

# 如有多張網卡，請將 public network 設定於 ceph.conf 文件的 [global] 段落下
# [global] 段
public network = {ip-address}/{netmask}

# 開始安裝 ceph 在各節點上
ceph-deploy install admin-node node1 node2 node3

# 如果需要重置復原(反安裝概念)，下指令
ceph-deploy purge admin-node node1 node2 node3

# 初始化 Monitor 服務，並收集各節點密鑰
ceph-deploy mon create-initial
# 完成後 會出現下列密鑰檔
{cluster-name}.client.admin.keyring
{cluster-name}.bootstrap-osd.keyring
{cluster-name}.bootstrap-mds.keyring
{cluster-name}.bootstrap-mgr.keyring
{cluster-name}.bootstrap-rgw.keyring

# 以ceph命令檢查密鑰:
ceph-deploy gatherkeys ceph-mon

# 透過 ceph-deploy 將配置與 admin密鑰傳送到管理節點與 OSD 節點，便於未來執行 Ceph 指令。
# ceph-deploy admin {admin-node} {ceph-node}
ceph-deploy admin admin-node node1 node2 node3

# 確保有此檔案的讀取權限
sudo chmod +r /etc/ceph/ceph.client.admin.keyring

# 部署 manager daemon
ceph-deploy mgr create node1
```

### 新增 OSD

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
peering 完成后，集群应该达到`active + clean`状态。  
OSD 目錄：/var/lib/ceph/osd/{osd.id}
{% endhint %}

{% hint style="info" %}
文章出處：  
[http://docs.ceph.org.cn/start/quick-ceph-deploy/\#id2](http://docs.ceph.org.cn/start/quick-ceph-deploy/#id2)
{% endhint %}

### 配置 OSD

```bash
[osd]
osd_journal = /tmp/ceph/journal
osd_journal_size = 5120
```

{% hint style="info" %}
[http://docs.ceph.com/docs/mimic/rados/configuration/osd-config-ref/](http://docs.ceph.com/docs/mimic/rados/configuration/osd-config-ref/)  
[http://docs.ceph.com/docs/mimic/rados/configuration/journal-ref/](http://docs.ceph.com/docs/mimic/rados/configuration/journal-ref/)
{% endhint %}

### 重啟/觀察 OSD 服務

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

### OSD in / out service

```bash
sudo ceph osd out|in {osd.id}
sudo ceph osd in 1
sudo ceph osd out 1
```

### Remove OSD

```bash
# in OSD node
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

### 監控 OSD

```bash
sudo ceph -w | -s
  -s, --status          show cluster status
  -w, --watch           watch live cluster changes

sudo ceph osd tree
```

[http://docs.ceph.com/docs/mimic/rados/configuration/mon-osd-interaction](http://docs.ceph.com/docs/mimic/rados/configuration/mon-osd-interaction)

### OSD Cache

Bluestore use bcache

