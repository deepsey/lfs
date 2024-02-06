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
![](lfs-001.jpg)

3. Создал директорию /mnt/lfs и systemd mount unit для монтирования в эту директорию логического тома lfs
```
mkdir /mnt/lfs
```
Содержимое юнита для монтирования `/etc/systemd/system/mnt-lfs.mount`:
```
Unit]
Description=/mnt/lfs mount point

[Mount]
What=/dev/mapper/vg0-lfs
Where=/mnt/lfs
Type=ext4
DirectoryMode=0777
Options=defaults


[Install]
WantedBy=multi-user.target
```
После создания юнита делаем 
```
systemctl daemon-reload
systemctl enable mnt-lfs.mount
systemctl start mnt-lfs.mount
systemctl status mnt-lfs.mount
```
Проверяем монтирование тома и затем меняем его владельца
```
chown vagrant:vagrant /mnt/lfs
```
4. Проверяем систему на наличие нужных пакетов для сборки. Для этого запускаем скрипт из книги
```
./version-check.sh
```
В моем случае пришлось доустановить пакет `texinfo` и изменить ссылку `/bin/sh`, так как она указывала на файл `/bin/dash` вместо `/bin/bash`
```
apt install texinfo
ln -sf /bin/bash /bin/sh
```

5. Немного не в тему - полезный пакет для поиска пакета, содержащего имя файла - `apt-file`. Ниже пример установки и использования команды для поиска пакета, содержащего `yacc`. 
```
apt install apt-file
apt-file update
apt-file search yacc | grep "yacc$"
```

nano wget-list
   21  wc -l wget-list 
   22  nano wget.sh  
   23  chmod +x wget.sh 
   24  mkdir sources
   25  ./wget.sh 
   26  ls 1 sources
   27  ls -1 sources
   28  ls -1 sources | sort > local_files
   29  cat local_files 
   30  awk -F"/" '{print $NF}' wget-list | sort > wget_files
   31  cat wget_files 
   32  diff local_files wget_files 
   33  cat wget-list | grep zlib
   34  wget https://zlib.net/zlib-1.2.11.tar.xz --continue --directory-prefix=$LFS/sources
   35  wget https://zlib.net/zlib-1.2.12.tar.xz --continue --directory-prefix=$LFS/sources
   36  wget https://zlib.net/zlib-1.2.13.tar.xz --continue --directory-prefix=$LFS/sources
   37  wget https://zlib.net/zlib-1.3.1.tar.xz --continue --directory-prefix=$LFS/sources
   38  cat wget-list | grep expat
   39  wget https://github.com/libexpat/libexpat/releases/tag/R_2_4_6/expat-2.4.6.tar.xz --continue --directory-prefix=$LFS/sources
   40  wc -l sources/
   41  ls -1 sources | sort > local_files
   42  wc -l local_files 
   43  diff local_files wget_files
6. 
   
