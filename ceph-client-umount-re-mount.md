# 問題：ceph client umount &gt; re mount

```bash
[afu@client-206 ~]$ sudo rbd map p_rbd/gitlab_bk --name client.admin -m 192.168.13.231 -k /etc/ceph/ceph.client.admin.keyring
rbd: sysfs write failed

^C
[afu@client-206 ~]$

[afu@client-206 ~]$ sudo dmesg |tail
[6419933.049356] EXT4-fs error (device rbd0): ext4_mb_generate_buddy:757: group 448, block bitmap and bg descriptor inconsistent: 23114 vs 24544 free clusters
[6420032.720851] EXT4-fs (rbd0): error count since last fsck: 38
[6420032.720865] EXT4-fs (rbd0): initial error at time 1542617684: ext4_mb_generate_buddy:757
[6420032.720870] EXT4-fs (rbd0): last error at time 1542621618: ext4_mb_generate_buddy:757
[6421122.200281] hpet1: lost 1 rtc interrupts
[6421722.259750] hpet1: lost 2 rtc interrupts
[6422322.192788] hpet1: lost 8 rtc interrupts
[6422922.161401] hpet1: lost 9 rtc interrupts
[6423522.159335] hpet1: lost 10 rtc interrupts
[6424122.420581] hpet1: lost 18 rtc interrupts
```

