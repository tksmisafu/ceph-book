# Ceph Metadata Server \(MDS\) Architecture

* MDSs
  * 提供Ceph文件系統存儲元數據。
  * Ceph 元數據服務器允許 POSIX 文件系統用戶執行基本命令（如ls、find等），而不會給 Ceph 存儲集群帶來巨大負擔。
  * CPU 需求稍高 8 核心、每进程 1GB。

![](.gitbook/assets/ceph-metadata-server-mds.png)

![](.gitbook/assets/image%20%282%29.png)

{% hint style="info" %}
文章參考：  
[http://docs.ceph.com/docs/giant/architecture/](http://docs.ceph.com/docs/giant/architecture/)  
[https://access.redhat.com/documentation/en-us/red\_hat\_ceph\_storage/3/html-single/ceph\_file\_system\_guide/index](https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/3/html-single/ceph_file_system_guide/index)
{% endhint %}





