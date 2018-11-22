# 建置 RBD with iSCSI

### overview

![](.gitbook/assets/image%20%285%29.png)

{% hint style="info" %}
參考文章：  
[http://docs.ceph.com/docs/mimic/rbd/iscsi-overview/](http://docs.ceph.com/docs/mimic/rbd/iscsi-overview/)
{% endhint %}

## Requirements

```bash
sudo yum install cmake git cmake libxslt-devel python-devel libnl3 kmod librbd1 glib2-devel libnl3-devel kmod-devel librbd1-devel librados2-devel
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && sudo python get-pip.py

vi requirments.txt
# 內容
    pyparsing
    kmod
    pyudev
    urwid
    netifaces
    crypto
    pycrypto
    requests
    flask
    pyOpenSSL
    netaddr

sudo pip install -r requirments.txt

# sudo yum install python-kmod python-pyudev python-gobject python-urwid python-rados python-rbd python-netaddr python-netifaces python-crypto python-requests python-flask pyOpenSSL
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
sudo systemctl daemon-reload
sudo systemctl enable tcmu-runner
sudo systemctl start tcmu-runner
```

#### RTSLIB-FB

```bash
git clone https://github.com/open-iscsi/rtslib-fb.git
cd rtslib-fb
sudo python setup.py install

# python module pyudev==0.15 > pyudev==0.21.0
# python module add rtslib-fb==2.1.69
```

#### CONFIGSHELL-FB

```bash
git clone https://github.com/open-iscsi/configshell-fb.git
cd configshell-fb
sudo python setup.py install

# python module add configshell-fb==1.1.25
```

#### TARGETCLI-FB

```bash
git clone https://github.com/open-iscsi/targetcli-fb.git
cd targetcli-fb
sudo python setup.py install
sudo mkdir /etc/target
sudo mkdir /var/target

# python module add targetcli-fb===2.1.fb49
```

#### CEPH-ISCSI

```bash
git clone https://github.com/ceph/ceph-iscsi.git
cd ceph-iscsi
sudo python setup.py install --install-scripts=/usr/bin
sudo cp usr/lib/systemd/system/rbd-target-gw.service /lib/systemd/system
sudo cp usr/lib/systemd/system/rbd-target-api.service /lib/systemd/system

# python module add  ceph-iscsi==3.0

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



