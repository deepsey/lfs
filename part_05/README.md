# Часть 5

## Chapter 7. Entering Chroot and Building Additional Temporary Tools

Подготавливаем каталог $LFS таким образом, чтобы он отвечал требванием каталога / обычной системы linux. Например, нам нужно создать каталоги /dev, /proc, /sys и подключить их как каталоги нашей реальной системы.

Далее мы будем работать от пользователя root, поэтому необходимо убедиться, что у этого пользователя прописана переменная $LFS. Определим переменную $LFS в файле .bashrc пользователя root или в файле /etc/profile
```
export $LFS=/mnt/lfs
```
export - переменная наследуется в дочерние процессы  
Меняем пользователя каталогов на root
```
chown -R root:root $LFS/{usr,lib,var,etc,bin,sbin,tools}
case $(uname -m) in
x86_64) chown -R root:root $LFS/lib64 ;;
esac
```
### 🔷 Подготавливаем виртуальные файловые системы

Создаем директории для файловых систем
```
mkdir -pv $LFS/{dev,proc,sys,run}
.................................
mkdir: created directory '/mnt/lfs/dev'
mkdir: created directory '/mnt/lfs/proc'
mkdir: created directory '/mnt/lfs/sys'
mkdir: created directory '/mnt/lfs/run'
```
### 🔷 Создаем файлы устройств

Создаем устройства console и null
```
mknod -m 600 $LFS/dev/console c 5 1
mknod -m 666 $LFS/dev/null c 1 3
....................................
# ls -l $LFS/dev/
total 0
crw------- 1 root root 5, 1 Feb 20 10:23 console
crw-rw-rw- 1 root root 1, 3 Feb 20 10:24 null
```

c 5 1 - мажор и минор числа. 5 - с драйвером какого устройства будет взаимодействовать этот файл, 1 - уникальный номер устройства. Посмотреть список наших мажорных номеров можно в файле /proc/devices
```
cat /proc/devices
.....................
Character devices:                                                                                                                                                                           
  1 mem                                                                                                                                                                                      
  4 /dev/vc/0                                                                                                                                                                                
  4 tty                                                                                                                                                                                      
  4 ttyS                                                                                                                                                                                     
  5 /dev/tty
.....................
```
Посмотреть мажоры и миноры для, например, символьных утсройств можно так
```
ls -l /sys/dev/char
```
### 🔷 Связываем каталоги хостовой системы с каталогами будущего chroot
Небольшое отступление: представим ситуацию - у нас есть каталог с содержимым, и в этот каталог примрнтировано какаое-то дисковое утсройство. Как увидеть исходное содержимое каталога без отмонтирования устройства? Для этого используется опция bind в команде mount. Например, от root примонтируем наше устройство в каталог /bind, в котором предварительно создадим файлы
```
mkdir /bind
cd /bind
touch file{1..10}
ls /bind
.....................................................................
file1  file10  file2  file3  file4  file5  file6  file7  file8  file9
......................................................................
cd
umount /mnt/lfs
mount /dev/mapper/vg0-lfs /bind
ls /bind
...........................................................................................
bin  dev  etc  lib  lib64  lost+found  proc  run  sbin  sources  sys  tmp  tools  usr  var
```
Видим, что теперь содержимое папки /bind соответсвует содержимому папки $LFS.  
Создадим папку /root/temp и примонтируем в нее хостовую файловую систему с опцией bind  
```
mount --bind / /root/temp
```
Теперь посмотрим содержимое папки /root/temp/bind
```
# ls /root/temp/bind
......................................................................
file1  file10  file2  file3  file4  file5  file6  file7  file8  file9
```
Видим исходное содержимое папки /bind.



