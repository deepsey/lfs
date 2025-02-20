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




