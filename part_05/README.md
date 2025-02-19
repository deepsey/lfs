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






