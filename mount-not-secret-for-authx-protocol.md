# 問題：mount not secret \(for auth\_x protocol\)

```bash
[afu@client-211 ~]$ sudo mount -t ceph 192.168.13.231:6789:/ /mnt/cephfs
mount: wrong fs type, bad option, bad superblock on 192.168.13.231:6789:/,
       missing codepage or helper program, or other error

       In some cases useful info is found in syslog - try
       dmesg | tail or so.
[afu@client-211 ~]$
[afu@client-211 ~]$ sudo dmesg | tail
[433482.125371] hpet1: lost 2 rtc interrupts
[434607.343494] Key type dns_resolver registered
[434607.376614] Key type ceph registered
[434607.376774] libceph: loaded (mon/osd proto 15/24)
[434607.409335] ceph: loaded (mds proto 32)
[434607.412467] libceph: no secret set (for auth_x protocol)
[434607.413036] libceph: error -22 on auth protocol 2 init
[434682.125300] hpet1: lost 2 rtc interrupts
[435051.186849] libceph: no secret set (for auth_x protocol)
[435051.187411] libceph: error -22 on auth protocol 2 init
[afu@client-211 ~]$

```

#### 上述問題，是很明確缺少驗證



