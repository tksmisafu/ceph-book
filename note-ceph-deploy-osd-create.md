# note: ceph-deploy osd create

```bash
[ceph-admin@dev-ceph-mon ceph-cluster]$ ceph-deploy osd create --data /dev/sdc dev-ceph-osd1
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/ceph-admin/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /bin/ceph-deploy osd create --data /dev/sdc dev-ceph-osd1
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  bluestore                     : None
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x7f87dd9a7d40>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  fs_type                       : xfs
[ceph_deploy.cli][INFO  ]  block_wal                     : None
[ceph_deploy.cli][INFO  ]  default_release               : False
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  journal                       : None
[ceph_deploy.cli][INFO  ]  subcommand                    : create
[ceph_deploy.cli][INFO  ]  host                          : dev-ceph-osd1
[ceph_deploy.cli][INFO  ]  filestore                     : None
[ceph_deploy.cli][INFO  ]  func                          : <function osd at 0x7f87ddbee848>
[ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[ceph_deploy.cli][INFO  ]  zap_disk                      : False
[ceph_deploy.cli][INFO  ]  data                          : /dev/sdc
[ceph_deploy.cli][INFO  ]  block_db                      : None
[ceph_deploy.cli][INFO  ]  dmcrypt                       : False
[ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[ceph_deploy.cli][INFO  ]  dmcrypt_key_dir               : /etc/ceph/dmcrypt-keys
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  debug                         : False
[ceph_deploy.osd][DEBUG ] Creating OSD on cluster ceph with data device /dev/sdc
[dev-ceph-osd1][DEBUG ] connection detected need for sudo
[dev-ceph-osd1][DEBUG ] connected to host: dev-ceph-osd1
[dev-ceph-osd1][DEBUG ] detect platform information from remote host
[dev-ceph-osd1][DEBUG ] detect machine type
[dev-ceph-osd1][DEBUG ] find the location of an executable
[ceph_deploy.osd][INFO  ] Distro info: CentOS Linux 7.5.1804 Core
[ceph_deploy.osd][DEBUG ] Deploying osd to dev-ceph-osd1
[dev-ceph-osd1][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
[dev-ceph-osd1][DEBUG ] find the location of an executable
[dev-ceph-osd1][INFO  ] Running command: sudo /usr/sbin/ceph-volume --cluster ceph lvm create --bluestore --data /dev/sdc
[dev-ceph-osd1][DEBUG ] Running command: /bin/ceph-authtool --gen-print-key
[dev-ceph-osd1][DEBUG ] Running command: /bin/ceph --cluster ceph --name client.bootstrap-osd --keyring /var/lib/ceph/bootstrap-osd/ceph.keyring -i - osd new 6da33502-315a-4394-83cb-507e6170f322
[dev-ceph-osd1][DEBUG ] Running command: /usr/sbin/vgcreate --force --yes ceph-2441c7cc-8403-48f3-84da-f1be2ce610d5 /dev/sdc
[dev-ceph-osd1][DEBUG ]  stdout: Physical volume "/dev/sdc" successfully created.
[dev-ceph-osd1][DEBUG ]  stdout: Volume group "ceph-2441c7cc-8403-48f3-84da-f1be2ce610d5" successfully created
[dev-ceph-osd1][DEBUG ] Running command: /usr/sbin/lvcreate --yes -l 100%FREE -n osd-block-6da33502-315a-4394-83cb-507e6170f322 ceph-2441c7cc-8403-48f3-84da-f1be2ce610d5
[dev-ceph-osd1][DEBUG ]  stdout: Logical volume "osd-block-6da33502-315a-4394-83cb-507e6170f322" created.
[dev-ceph-osd1][DEBUG ] Running command: /bin/ceph-authtool --gen-print-key
[dev-ceph-osd1][DEBUG ] Running command: /bin/mount -t tmpfs tmpfs /var/lib/ceph/osd/ceph-1
[dev-ceph-osd1][DEBUG ] Running command: /usr/sbin/restorecon /var/lib/ceph/osd/ceph-1
[dev-ceph-osd1][DEBUG ] Running command: /bin/chown -h ceph:ceph /dev/ceph-2441c7cc-8403-48f3-84da-f1be2ce610d5/osd-block-6da33502-315a-4394-83cb-507e6170f322
[dev-ceph-osd1][DEBUG ] Running command: /bin/chown -R ceph:ceph /dev/dm-3
[dev-ceph-osd1][DEBUG ] Running command: /bin/ln -s /dev/ceph-2441c7cc-8403-48f3-84da-f1be2ce610d5/osd-block-6da33502-315a-4394-83cb-507e6170f322 /var/lib/ceph/osd/ceph-1/block
[dev-ceph-osd1][DEBUG ] Running command: /bin/ceph --cluster ceph --name client.bootstrap-osd --keyring /var/lib/ceph/bootstrap-osd/ceph.keyring mon getmap -o /var/lib/ceph/osd/ceph-1/activate.monmap
[dev-ceph-osd1][DEBUG ]  stderr: got monmap epoch 1
[dev-ceph-osd1][DEBUG ] Running command: /bin/ceph-authtool /var/lib/ceph/osd/ceph-1/keyring --create-keyring --name osd.1 --add-key AQCyBthb+NXLOhAAcDwcKuJ+5c1FytHOfJs5pA==
[dev-ceph-osd1][DEBUG ]  stdout: creating /var/lib/ceph/osd/ceph-1/keyring
[dev-ceph-osd1][DEBUG ] added entity osd.1 auth auth(auid = 18446744073709551615 key=AQCyBthb+NXLOhAAcDwcKuJ+5c1FytHOfJs5pA== with 0 caps)
[dev-ceph-osd1][DEBUG ] Running command: /bin/chown -R ceph:ceph /var/lib/ceph/osd/ceph-1/keyring
[dev-ceph-osd1][DEBUG ] Running command: /bin/chown -R ceph:ceph /var/lib/ceph/osd/ceph-1/
[dev-ceph-osd1][DEBUG ] Running command: /bin/ceph-osd --cluster ceph --osd-objectstore bluestore --mkfs -i 1 --monmap /var/lib/ceph/osd/ceph-1/activate.monmap --keyfile - --osd-data /var/lib/ceph/osd/ceph-1/ --osd-uuid 6da33502-315a-4394-83cb-507e6170f322 --setuser ceph --setgroup ceph
[dev-ceph-osd1][DEBUG ] --> ceph-volume lvm prepare successful for: /dev/sdc
[dev-ceph-osd1][DEBUG ] Running command: /bin/ceph-bluestore-tool --cluster=ceph prime-osd-dir --dev /dev/ceph-2441c7cc-8403-48f3-84da-f1be2ce610d5/osd-block-6da33502-315a-4394-83cb-507e6170f322 --path /var/lib/ceph/osd/ceph-1 --no-mon-config
[dev-ceph-osd1][DEBUG ] Running command: /bin/ln -snf /dev/ceph-2441c7cc-8403-48f3-84da-f1be2ce610d5/osd-block-6da33502-315a-4394-83cb-507e6170f322 /var/lib/ceph/osd/ceph-1/block
[dev-ceph-osd1][DEBUG ] Running command: /bin/chown -h ceph:ceph /var/lib/ceph/osd/ceph-1/block
[dev-ceph-osd1][DEBUG ] Running command: /bin/chown -R ceph:ceph /dev/dm-3
[dev-ceph-osd1][DEBUG ] Running command: /bin/chown -R ceph:ceph /var/lib/ceph/osd/ceph-1
[dev-ceph-osd1][DEBUG ] Running command: /bin/systemctl enable ceph-volume@lvm-1-6da33502-315a-4394-83cb-507e6170f322
[dev-ceph-osd1][DEBUG ]  stderr: Created symlink from /etc/systemd/system/multi-user.target.wants/ceph-volume@lvm-1-6da33502-315a-4394-83cb-507e6170f322.service to /usr/lib/systemd/system/ceph-volume@.service.
[dev-ceph-osd1][DEBUG ] Running command: /bin/systemctl enable --runtime ceph-osd@1
[dev-ceph-osd1][DEBUG ] Running command: /bin/systemctl start ceph-osd@1
[dev-ceph-osd1][DEBUG ] --> ceph-volume lvm activate successful for osd ID: 1
[dev-ceph-osd1][DEBUG ] --> ceph-volume lvm create successful for: /dev/sdc
[dev-ceph-osd1][INFO  ] checking OSD status...
[dev-ceph-osd1][DEBUG ] find the location of an executable
[dev-ceph-osd1][INFO  ] Running command: sudo /bin/ceph --cluster=ceph osd stat --format=json
[ceph_deploy.osd][DEBUG ] Host dev-ceph-osd1 is now ready for osd use.
[ceph-admin@dev-ceph-mon ceph-cluster]$
```

