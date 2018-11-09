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
ssh-keyscan ceph-mon1 ceph-osd1 ceph-osd2 ceph-osd3 >> ~/.ssh/known_hosts
ssh-keygen
ssh-copy-id {username}@node1
ssh-copy-id {username}@node2
ssh-copy-id {username}@node3
```

#### 開設網路存取權 && 設定 Firewalld

```bash
# ??
ssh ceph-mon1 'sudo firewall-cmd --zone=public --add-port=6789/tcp --permanent && sudo firewall-cmd --reload'

# on Monitor node
ssh ceph-mon-232 'sudo firewall-cmd --zone=public --add-service=ceph-mon --permanent && sudo firewall-cmd --reload'

# on OSD node
ssh ceph-osd-1 'sudo firewall-cmd --zone=public --add-service=ceph --permanent && sudo firewall-cmd --reload'

```

### 創建集群

基於管理方便，先在管理節點上創建一個目錄，用於保存 ceph-deploy 所產出的配置文件與密鑰。並且在後續執行 ceph-deploy 時，是在此新目錄內下指令。

```bash
mkdir my-cluster
cd my-cluster

# 1.Create the cluster 初始創建集群，在 Monitor 節點執行指令（initial-monitor-node）
ceph-deploy new node1
# 上述執行後，會產出三份檔案：
# ceph.conf 配置文件、
# ceph.mon.keyring 密鑰 和 
# ceph-deploy-ceph.log 日誌文件。

# 2.Install Ceph packages 開始安裝 ceph 在各節點上
ceph-deploy install admin-node node1 node2 node3
# 此步驟在 admin-node 上出現錯誤：RuntimeError: NoSectionError: No section: 'ceph'
# 進行除錯：yum remove ceph-release && rm /etc/yum.repos.d/ceph.repo
# 再重新 ceph-deploy install 即可正常安裝。
# 安裝成功後訊息：
[ceph-osd-1][DEBUG ] Complete!
[ceph-osd-1][INFO  ] Running command: sudo ceph --version
[ceph-osd-1][DEBUG ] ceph version 13.2.2 (02899bfda814146b021136e9d8e80eba494e1126) mimic (stable)

# Deploy the initial monitor(s) and gather the keys
# 初始化 Monitor 服務，並收集各節點密鑰
ceph-deploy mon create-initial
# 完成後 會出現下列密鑰檔
{cluster-name}.client.admin.keyring
{cluster-name}.bootstrap-osd.keyring
{cluster-name}.bootstrap-mds.keyring
{cluster-name}.bootstrap-mgr.keyring
{cluster-name}.bootstrap-rgw.keyring

# 以ceph命令檢查密鑰:
# ceph-deploy gatherkeys ceph-mon
# 執行結果：[ceph_deploy][ERROR ] RuntimeError: connecting to host: ceph-mon resulted in errors: HostNotFound ceph-mon

# 透過 ceph-deploy 將配置與 admin密鑰傳送到管理節點與 OSD 節點，便於未來執行 Ceph 指令。
# ceph-deploy admin {admin-node} {ceph-node}
ceph-deploy --overwrite-conf admin admin-node1 {admin-node2}

# 確保有此檔案的讀取權限
sudo chmod +r /etc/ceph/ceph.client.admin.keyring

# 部署 manager daemon
ceph-deploy mgr create node1
```

```bash
# 如模擬兩個OSD環境下，設定副本數可以呈現 “active + clean” 系統狀態
# ceph.conf 文件內 [global] 段落
osd pool default size = 2

# 如有多張網卡，請將 public network 設定於 ceph.conf 文件的 [global] 段落下
# [global] 段
public network = {ip-address}/{netmask}

```

```bash

# 如果需要重置復原(反安裝概念)，下指令
ceph-deploy purge admin-node node1 node2 node3
```

{% hint style="info" %}
文章出處：  
[http://docs.ceph.com/docs/mimic/start/quick-start-preflight/](http://docs.ceph.com/docs/mimic/start/quick-start-preflight/)  
[http://docs.ceph.com/docs/mimic/start/quick-ceph-deploy/\#create-a-cluster](http://docs.ceph.com/docs/mimic/start/quick-ceph-deploy/#create-a-cluster)
{% endhint %}

