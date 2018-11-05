# Pool & Placement Group 配置

觀念篇

當你新增 pool 時，就要指定 pg\_num ~ [官網文件](http://docs.ceph.com/docs/master/rados/operations/placement-groups/)這樣建議：

* Less than 5 OSDs set `pg_num` to 128
* Between 5 and 10 OSDs set `pg_num` to 512
* Between 10 and 50 OSDs set `pg_num` to 1024
* If you have more than 50 OSDs, you need to understand the tradeoffs and how to calculate the `pg_num` value by yourself
* For calculating `pg_num` value by yourself please take help of [pgcalc](http://ceph.com/pgcalc/) tool

PG 數量，是基於 OSD、pool replica 等因素計算出來  
~~例如 20 OSD、pg\_num 1024、pool replica 3~~  
~~則計算方式~~ $$1024*3/20=153.6$$ ~~Placement Groups~~

官方 PG 計算工具：[https://ceph.com/pgcalc/](https://ceph.com/pgcalc/)

  


