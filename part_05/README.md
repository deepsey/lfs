# Часть 5

## Chapter 7. Entering Chroot and Building Additional Temporary Tools

Подготавливаем каталог $LFS таким образом, чтобы он отвечал требванием каталога / обычной системы linux. Например, нам нужно создать каталоги /dev, /proc, /sys и подключить их как каталоги нашей реальной системы.

Далее мы будем работать от пользователя root, поэтому необходимо убедиться, что у этого пользователя прописана переменная $LFS. Определим переменную $LFS в файле .bashrc пользователя root или в файле /etc/profile
```
export LFS=/mnt/lfs
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
### 🔷 Связываем каталог /dev хостовой системы с каталогом /dev будущего chroot
Небольшое отступление: представим ситуацию - у нас есть каталог с содержимым, и в этот каталог примонтировано какое-то дисковое утсройство. Как увидеть исходное содержимое каталога без отмонтирования устройства? Для этого используется опция bind в команде mount. Например, от root примонтируем наше устройство в каталог /bind, в котором предварительно создадим файлы
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

---  

  
Связываем каталог хостовой системы /dev с каталогом /$LFS/dev
```
mount -v --bind /dev $LFS/dev
```
Монтируем виртуальные корневые файловые сиситемы
```
mount -v --bind /dev/pts $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run
```
В некоторых системах /dev/shm представляет собой символическую ссылку на /run/shm. Временная файловая система уже была смонтирована выше, поэтому в этом случае необходимо лишь создать директорию
```
if [ -h $LFS/dev/shm ]; then
   mkdir -pv $LFS/$(readlink $LFS/dev/shm)
fi
```
### 🔷 Заходим в окружение chroot
```
chroot "$LFS" /usr/bin/env -i HOME=/root TERM="$TERM" PS1='(lfs chroot) \u:\w\$ ' PATH=/usr/bin:/usr/sbin /bin/bash --login +h
...............................................................................................................................
(lfs chroot) I have no name!:/# 
```
/usr/bin/env - устанавливает переменные окружения  
PS1='(lfs chroot) - переменная, определяющая строку приглашения

### 🔷 Создаем необходимые директории
```
mkdir -pv /etc/{opt,sysconfig}
mkdir -pv /lib/firmware
mkdir -pv /media/{floppy,cdrom}
mkdir -pv /usr/{,local/}{include,src}
mkdir -pv /usr/local/{bin,lib,sbin}
mkdir -pv /usr/{,local/}share/{color,dict,doc,info,locale,man}
mkdir -pv /usr/{,local/}share/{misc,terminfo,zoneinfo}
mkdir -pv /usr/{,local/}share/man/man{1..8}
mkdir -pv /var/{cache,local,log,mail,opt,spool}
mkdir -pv /var/lib/{color,misc,locate}

ln -sfv /run /var/run
ln -sfv /run/lock /var/lock

install -dv -m 0750 /root
install -dv -m 1777 /tmp /var/tmp
```
Команда install создает директорию и сразу дает ей нужные права для каталога /root, а в лучае с каталогом /tmp копирует его содержимое в /var/tmp и создает необходимые права.

### 🔷 Создаем необходимые файлы и симлинки
Исторически Linux содержит список монтируемых файловых систем в /etc/mtab. Современные ядра хранят этот список во внутренних структурах и делают его доступным для пользователя через файловую систему /proc. Для удовлетворения зависимостей создадим линк
```
ln -sv /proc/self/mounts /etc/mtab
```
Создадим файл /etc/hosts
```
cat > /etc/hosts << EOF
127.0.0.1 localhost $(hostname)
::1 localhost
EOF
```
Для логина пользователя root и распознавания его имени добавим необходимые параметры в файлы /etc/passwd и /etc/group.
Создаем /etc/passwd
```
cat > /etc/passwd << "EOF"
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/dev/null:/bin/false
daemon:x:6:6:Daemon User:/dev/null:/bin/false
messagebus:x:18:18:D-Bus Message Daemon User:/run/dbus:/bin/false
uuidd:x:80:80:UUID Generation Daemon User:/dev/null:/bin/false
nobody:x:99:99:Unprivileged User:/dev/null:/bin/false
EOF
```
Создаем /etc/group
```
cat > /etc/group << "EOF"
root:x:0:
bin:x:1:daemon
sys:x:2:
kmem:x:3:
tape:x:4:
tty:x:5:
daemon:x:6:
floppy:x:7:
disk:x:8:
lp:x:9:
dialout:x:10:
audio:x:11:
video:x:12:
utmp:x:13:
usb:x:14:
cdrom:x:15:
adm:x:16:
messagebus:x:18:
input:x:24:
mail:x:34:
kvm:x:61:
uuidd:x:80:
wheel:x:97:
nogroup:x:99:
users:x:999:
EOF
```
Некоторые тесты в дальнейшем требуют наличия регулярного пользователя. Создадим его, а в дальнейше, когда необходимость в нем пропадет, удалим.
```
echo "tester:x:101:101::/home/tester:/bin/bash" >> /etc/passwd
echo "tester:x:101:" >> /etc/group
install -o tester -d /home/tester
```
Для того, чтобы убрать строку "I have no name!", перезапускаем оболочку
```
exec /bin/bash --login +h
```
Директива +h заставляет bash использовать внутренние пути для хэширования. Без этой директивы bash запоминает пути до ранее выполненных бинарников.  

Инициализируем лог-файлы и даем им нужные права
```
touch /var/log/{btmp,lastlog,faillog,wtmp}
chgrp -v utmp /var/log/lastlog
chmod -v 664 /var/log/lastlog
chmod -v 600 /var/log/btmp
```
/var/log/wtmp - записывает все входы и выходы из системы  
/var/log/lastlog - содержит записи о времени последнего входа каждого пользователя  
/var/log/faillog - записывает неудачные попытки входа  
/var/log/btmp - записывает вредоносные попытки входа  

/run/utmp - записывает пользователей, которые залогинены в текущий момент. Этот файл формируется динамически загрузочными скриптами.  




