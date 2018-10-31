# Ceph 有哪些角色

![Architecture](.gitbook/assets/stack.png)



#### A Ceph Storage Cluster consists of two types of daemons:

* [Ceph Monitor](http://docs.ceph.com/docs/master/glossary/#term-ceph-monitor)
  * Ceph Monitor maintains a master copy of the cluster map. 
  * A cluster of Ceph monitors ensures high availability should a monitor daemon fail. 
  * Storage cluster clients retrieve a copy of the cluster map from the Ceph Monitor.
* [Ceph OSD Daemon](http://docs.ceph.com/docs/master/glossary/#term-ceph-osd-daemon)
  * Ceph OSD Daemons handle the read/write operations on the storage disks.



### Ceph Clients include a number of service interfaces. These include:

* **Block Devices:** The [Ceph Block Device](http://docs.ceph.com/docs/master/glossary/#term-ceph-block-device) \(a.k.a., RBD\) service provides resizable, thin-provisioned block devices with snapshotting and cloning. Ceph stripes a block device across the cluster for high performance. Ceph supports both kernel objects \(KO\) and a QEMU hypervisor that uses `librbd` directly–avoiding the kernel object overhead for virtualized systems.
* **Object Storage:** The [Ceph Object Storage](http://docs.ceph.com/docs/master/glossary/#term-ceph-object-storage) \(a.k.a., RGW\) service provides RESTful APIs with interfaces that are compatible with Amazon S3 and OpenStack Swift.
* **Filesystem**: The [Ceph Filesystem](http://docs.ceph.com/docs/master/glossary/#term-ceph-filesystem) \(CephFS\) service provides a POSIX compliant filesystem usable with `mount` or as a filesystem in user space \(FUSE\).



參考：  
[http://docs.ceph.com/docs/master/architecture/](http://docs.ceph.com/docs/master/architecture/)  


