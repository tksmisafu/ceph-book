# 問題：Mount 出現 hpet 錯誤訊息

```bash
vi /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT="quiet hpet=disable"

sudo grub2-mkconfig -o /boot/grub2/grub.cfg

sudo reboot
```

