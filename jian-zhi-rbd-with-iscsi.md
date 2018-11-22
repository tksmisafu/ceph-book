# 建置 RBD with iSCSI

### overview

![](.gitbook/assets/image%20%285%29.png)

{% hint style="info" %}
參考文章：  
[http://docs.ceph.com/docs/mimic/rbd/iscsi-overview/](http://docs.ceph.com/docs/mimic/rbd/iscsi-overview/)
{% endhint %}

## Requirements

```bash
sudo yum install cmake git cmake libnl3 kmod librbd1 glib2-devel libnl3-devel kmod-devel librbd1-devel librados2-devel
sudo yum install python-kmod python-pyudev python-gobject python-urwid python-rados python-rbd python-netaddr python-netifaces python-crypto python-requests python-flask pyOpenSSL
```

#### TCMU-RUNNER

```bash
# tcmu-runner
# https://github.com/open-iscsi/tcmu-runner
# Installation:
git clone https://github.com/open-iscsi/tcmu-runner
cd tcmu-runner
./extra/install_dep.sh
cmake -Dwith-glfs=false -Dwith-qcow=false -DSUPPORT_SYSTEMD=ON -DCMAKE_INSTALL_PREFIX=/usr
#make
sudo make install

#sudo mkdir /etc/tcmu/
#sudo cp tcmu.conf /etc/tcmu/tcmu.conf
sudo vi /etc/tcmu/tcmu.conf
    log_level = 3
    log_dir_path = "/var/log"

# Enable and start the daemon:
systemctl daemon-reload
systemctl enable tcmu-runner
systemctl start tcmu-runner
```

#### RTSLIB-FB

```bash
git clone https://github.com/open-iscsi/rtslib-fb.git
cd rtslib-fb
sudo python setup.py install
```

#### CONFIGSHELL-FB

```bash
git clone https://github.com/open-iscsi/configshell-fb.git
cd configshell-fb
sudo python setup.py install
```

#### TARGETCLI-FB

```bash
git clone https://github.com/open-iscsi/targetcli-fb.git
cd targetcli-fb
sudo python setup.py install
sudo mkdir /etc/target
sudo mkdir /var/target
```

#### CEPH-ISCSI

```bash
git clone https://github.com/ceph/ceph-iscsi.git
cd ceph-iscsi
sudo python setup.py install --install-scripts=/usr/bin
sudo cp usr/lib/systemd/system/rbd-target-gw.service /lib/systemd/system
sudo cp usr/lib/systemd/system/rbd-target-api.service /lib/systemd/system

sudo systemctl daemon-reload
sudo systemctl enable rbd-target-gw
sudo systemctl start rbd-target-gw
sudo systemctl enable rbd-target-api
sudo systemctl start rbd-target-api

```

{% hint style="info" %}
iSCSI GW 服務安裝：  
[http://docs.ceph.com/docs/master/rbd/iscsi-target-cli-manual-install/](http://docs.ceph.com/docs/master/rbd/iscsi-target-cli-manual-install/)
{% endhint %}

## Configuring the iSCSI Target

## Configuring the iSCSI Initiators

## Monitoring the iSCSI Gateways



