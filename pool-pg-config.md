# Pool & Placement Group 配置

觀念篇

當你新增 pool 時，就要指定 pg\_num ~ [官網文件](http://docs.ceph.com/docs/master/rados/operations/placement-groups/)這樣建議：

* Less than 5 OSDs set `pg_num` to 128
* Between 5 and 10 OSDs set `pg_num` to 512
* Between 10 and 50 OSDs set `pg_num` to 1024
* If you have more than 50 OSDs, you need to understand the tradeoffs and how to calculate the `pg_num` value by yourself
* For calculating `pg_num` value by yourself please take help of [pgcalc](http://ceph.com/pgcalc/) tool

PG 數量，是基於 OSD、pool replica 等因素計算出來  
例如 20 OSD、pg\_num 1024、pool replica 3  
則計算方式 $$1024*3/20=153.6$$ PG per OSD

官方 PG 計算工具：[https://ceph.com/pgcalc/](https://ceph.com/pgcalc/)

對於設置多少的 PG 才是合理的，我從非公式的概念去想

```text
變數  PG、OSD，當 replica = 3 固定值
PG = 10     OSD = 10   PG-per-OSD=10*3/10=3，意思是當一個OSD掛了，損失 3/30 PG
PG = 20     OSD = 10   PG-per-OSD=20*3/10=6，意思是當一個OSD掛了，損失 6/60 PG
PG = 60     OSD = 10   PG-per-OSD=60*3/10=18，意思是當一個OSD掛了，損失 18/180 PG

變數  PG、OSD，當 replica = 3 固定值
PG = 10     OSD = 20   PG-per-OSD=10*3/20=1.5，意思是當一個OSD掛了，損失 1.5/30 PG
PG = 20     OSD = 20   PG-per-OSD=20*3/20=3，意思是當一個OSD掛了，損失 3/60 PG
PG = 60     OSD = 20   PG-per-OSD=60*3/20=9，意思是當一個OSD掛了，損失 9/180 PG
```

以上，當PG、replica 變數是固定的，新增 OSD 是有利降低損失風險

```text
變數  PG，當 OSD = 20、replica = 3 固定值
PG = 10     OSD = 20   PG-per-OSD=10*3/20=1.5，意思是當一個OSD掛了，損失 1.5/30 PG
PG = 20     OSD = 20   PG-per-OSD=20*3/20=3，意思是當一個OSD掛了，損失 3/60 PG
PG = 60     OSD = 20   PG-per-OSD=60*3/20=9，意思是當一個OSD掛了，損失 9/180 PG

變數  PG，當 OSD = 20、replica = 3 固定值
PG = 100    OSD = 20   PG-per-OSD=100*3/20=1.5，意思是當一個OSD掛了，損失 15/30 PG
PG = 200    OSD = 20   PG-per-OSD=200*3/20=3，意思是當一個OSD掛了，損失 30/60 PG
PG = 600    OSD = 20   PG-per-OSD=600*3/20=9，意思是當一個OSD掛了，損失 90/180 PG
```

以上，當 OSD、replica 變數是固定的，新增 PG 看似是拉高損失風險  
但有個新疑惑，一個 PG 代表著多少資料量？

