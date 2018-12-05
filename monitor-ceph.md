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



