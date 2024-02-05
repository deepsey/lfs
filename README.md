## Linux from scratch
1. Создал VM на Debian 11 с дополнительным HDD на 40GB
2. На дополнительном HDD создал LVM-раздел на 20GB и создал на нем ext4
``` 
apt install lvm2
pvcreate /dev/sdb
vgcreate vg0 /dev/sdb
lvcreate -L 20G -n lfs vg0
mkfs.ext4 /dev/vg0/lfs
vgdisplay vg0
lvdisplay /dev/vg0/lfs
```
Последние две команды - для просмотра созданных VolumeGroup и LogicalVolume. Для напоминания команды для вывода краткой информации по LVM
```
pvs | vgs | lvs
```

   20  
   21  lvs
   22  fdisk -l
   23  lsblk
   24  mkfs
   25  mkfs --help
   26  man mkfs
   27  mkfs.ext4 /vg0-lfs
   28  mkfs.ext4 /dev//vg0-lfs
   29  
   30  lvs
   31  vgscan
   32  lvmdiskscan
   33  vgdisplay lfs
   34  vgdisplay vg0
   35  lvdisplay /dev/vg0/lfs
   36  mkfs.ext4 /dev/vg0/lfs
   37  fdisk -l
   38  dpkg -l | grep binutils
   39  clear
   40  nano version-check.sh
   41  nano version-check.sh
   42  chmod +x version-check.sh 
   43  ./version-check.sh 
   44  apt install makeinfo
   45  apt install textinfo
   46  apt install texinfo
   47  ./version-check.sh 
   48  which bash
   49  ls /bin/bash
   50  ln -sf /bin/bash /bin/sh
   51  ./version-check.sh 
   52  mkdir /mnt/lfs
   53  chown vagrant:vagrant /mnt/lfs
   54  nano /etc/systemd/system/mnt-lfs.mount
   55  fdisk -l
   56  nano /etc/systemd/system/mnt-lfs.mount
   57  mount /dev/mapper/vg0-lfs /mnt/lfs
   58  cd /mnt/lfs/
   59  ls -la
   60  lsblk
   61  mkdir temp
   62  ls -la
   63  reboot
   64  cd /mnt/lfs
   65  ls
   66  systemctl daemon-reload
   67  systemctl enable mnt-lfs.mount
   68  systemctl start mnt-lfs.mount
   69  systemctl status mnt-lfs.mount
   70  ls
   71  ls -a
   72  lsblk
   73  mount /dev/mapper/vg0-lfs /mnt/lfs
   74  ls
   75  umount /dev/mapper/vg0-lfs
   76  ls
   77  mount /dev/mapper/vg0-lfs /mnt/lfs
   78  ls
   79  umount /dev/mapper/vg0-lfs
   80  systemctl restart' mnt-lfs.mount
   81  systemctl restart mnt-lfs.mount
   82  umount /dev/mapper/vg0-lfs
   83  lsblk
   84  systemctl restart mnt-lfs.mount
   85  lsblk
   86  ls -a
   87  reboot
   88  cd /mnt/lfs/
   89  ls
   90  lsblk
   91  mount
   92  nano /etc/systemd/system/mnt-lfs.mount 
   93  systemctl mon-reload
   94  systemctl restart mnt-lfs.mount
   95  journalctl -xe
   96  umount /dev/mapper/vg0-lfs
   97  cd
   98  umount /dev/mapper/vg0-lfs
   99  systemctl restart mnt-lfs.mount
  100  ls /mnt/lfs
  101  ls -l /mnt/
  102  reboot
  103  ls -l /mnt/
  104  cd /mnt/lfs
  105  mkdir 1111
  106  ls -l
  107  nano /etc/systemd/system/mnt-lfs.mount 
  108  systemctl daemon-reload
  109  cd
  110  umoumt /mnt/lfs/
  111  unmoumt /mnt/lfs/
  112  umount /mnt/lfs/
  113  systemctl restart mnt-lfs.mount
  114  ls -l /mnt
  115  cd /mnt/lfs/
  116  mkdir 22222
  117  ls -l
  118  chown -r vagrant:vagrant /mnt/lfs
  119  chown -R vagrant:vagrant /mnt/lfs
  120  ls -la
  121  reboot
  122  ls -l /mnt/lfs
  123  cd /mnt/lfs
  124  mkdir 3333
  125  ls -l /mnt/lfs
  126  ls -l /mnt
  127  exit
  128  apt install apt-file
  129  apt-file update
  130  app-file search yacc
  131  apt-file search yacc
  132  apt-file search yacc | grep "yacc$"
  133  history
