# 監控 Ceph

## Zabbix 監控 Ceph

### Zabbix Server 端

**Zabbix admin GUI** 上傳 template file  
檔案來源：**zabbix-agent** 端，檔案路徑：`/usr/lib64/ceph/mgr/zabbix/zabbix_template.xml`

### Zabbix Client 端

確認已經安裝 zabbix 套件  
`sudo yum install zabbix-agent zabbix-sender`

查看 Module info  
`sudo ceph mgr module ls`  
如果尚未 Enable，則需要啟用模組  
`sudo ceph mgr module enable zabbix`

```text
[ceph-admin@dev-ceph-mon afu]$ sudo ceph mgr module ls
{
    "enabled_modules": [
        "balancer",
        "iostat",
        "restful",
        "status",
        "zabbix"
    ],
    "disabled_modules": [
        {
            "name": "dashboard",
            "can_run": true,
            "error_string": ""
        }
    ]
}
```

參數設定

```text
ceph zabbix config-set <key> <value>
ceph zabbix config-set zabbix_host zabbix.server.ip
ceph zabbix config-set identifier [zabbix-Host name] | hostname
```

手動發送資料給 Zabbix-server

```text
# 方式一
[ceph-admin@dev-ceph-mon afu]$ ceph zabbix send
Sending data to Zabbix
# 方式二
zabbix_sender -vv -z [zabbix.server.ip] -s [zabbix.Host name] -k [Item.Key] -o 111

[ceph-admin@dev-ceph-mon afu]$ zabbix_sender -vv -z 10.1.13.234 -s dev-ceph-mon.kingbay-tech.com -k ceph.osd_avg_pgs -o 111
zabbix_sender [2959342]: DEBUG: answer [{"response":"success","info":"processed: 1; failed: 0; total: 1; seconds spent: 0.000055"}]
info from server: "processed: 1; failed: 0; total: 1; seconds spent: 0.000055"
sent: 1; skipped: 0; total: 1

```

{% hint style="info" %}
參考連結：  
[http://docs.ceph.com/docs/master/mgr/zabbix/](http://docs.ceph.com/docs/master/mgr/zabbix/)
{% endhint %}

## Prometheus 監控 Ceph

### Ceph mon 端

設定

```bash
# 設定參數
ceph config set mgr mgr/prometheus/server_addr 192.168.100.170
ceph config set mgr mgr/prometheus/server_port 9283

# 啟用模組
ceph mgr module enable prometheus

# 查看參數
sudo ceph config-key get config/mgr/mgr/prometheus/server_addr
```

觀察

```bash
sudo netstat -ntlp |grep 9283
iptables -L -n

```

### Prometheus server 端

設定

```text
vi prometheus.yml
# 新增
  - job_name: 'ceph'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['192.168.100.170:9283']
      
設定後，重啟 prometheus service
```



