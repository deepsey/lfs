# Linux from scratch

## <a name='000'>Содержание</a>  
[PART 01](#01)   
[PART 02](/../../tree/main/part_02#readme)   
[PART 03](/../../tree/main/part_03#readme)  
[PART 04](/../../tree/main/part_04#readme)     
[PART 05](/../../tree/main/part_05#readme)   
[PART 06](/../../tree/main/part_06#readme)   
[PART 07](/../../tree/main/part_07#readme)   
[PART 08](#08)   
[PART 09](#09)   
[PART 10](#10)   

---

##  <a name='01'>PART 01</a> 
1. Создал VM на Debian 11 (Ubuntu 22.04) с дополнительным HDD на 40GB
2. На дополнительном HDD создал LVM-раздел на 20GB и создал на нем ext4
``` 
apt install lvm2
pvcreate /dev/sdb
vgcreate vg0 /dev/sdb
lvcreate -L 20G -n lfs vg0 / lvcreate -l 100%FREE  -n lfs vg0
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
[Unit]
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
[Содержание](#000) 

---

##  <a name='02'>PART 02</a> 

🔷 1. Установим для пользователя переменную $LFS. В файл ~/.bashrc пропишем
```
export LFS=/mnt/lfs
```
🔷 2. Скачиваем необходимые пакеты. Список можно взять отсюда: https://www.linuxfromscratch.org/lfs/downloads/11.1/wget-list  
🔷 3. Подсчитаем количество пакетов для скачивания:
```
wc -l wget-list
93 wget-list
```
🔷 4. Создадим скрипт для скачивания пакетов `wget.sh`:
```
#!/bin/bash
wget --input-file=wget-list --continue --directory-prefix=$LFS/sources
```
`--input-file` - файл со списком для закачки
`--continue` - продолжать закачку приобрыве
`--directory-prefix` - целевая дитектория для скачиваемых файлов  

🔷 5. Создадим целевую директорию
```
mkdir $LFS/sources
```
🔷 6. Запускаем скрипт `wget.sh` и проверяем количество файлов в директории `$LFS/sources`
```
ls -1 $LFS/sources | wc -l
91
```
---
Можно запустить закачку файлов в несколько потоков, используя запуск процессов в фоновом режиме. Для этого можно запустить простой скрипт:
```
for i in `cat wget-list`; do wget $i --continue --directory-prefix=$LFS/sources & done
```
--- 
Видим, что какие-то два файла не скачались. Как нам их найти? Скопируем содержимое директории `$LFS/sources` в файл `local_files` с предварительной сортировкой строк
```
ls -1 sources | sort > local_files
```
Теперь создадим список файлов, которые мы должны были получить при скачивании из `wget-list`. Для этого воспользуемся командой `awk`
```
awk -F"/" '{print $NF}' wget-list | sort > wget_files
```
-F"/" - определяем разделитель полей, таким образом мы можем разбить каждую строчку wget-list на отдельные элементы, разделяемые слэшем  
'{print $NF}' - вывести последний ($NF) элемент строки

Весь вывод команды мы перенаправляем в файл `wget_files`, получив в нем отсортированный полный список файлов, которые мы должны были получить. Теперь мы можем сравнить два файла и понять, какие пакеты у нас не скачались:
```
diff local_files wget_files
```
В итоге не скачалось два архива: zlib-1.2.11.tar.xz и expat-2.4.6.tar.xz. Вместо zlib-1.2.11.tar.xz был скачен zlib-1.3.1.tar.xz, а expat-2.4.6.tar.xz был закачен из другого источника.
```
wget https://zlib.net/zlib-1.3.1.tar.xz --continue --directory-prefix=$LFS/sources
wget https://github.com/libexpat/libexpat/releases/tag/R_2_4_6/expat-2.4.6.tar.xz --continue --directory-prefix=$LFS/sources
```
После повтрения процедуры сравнения списка необходимых и скаченных файлов убедились, что теперь все необходимые архивы у нас есть.

🔷 7. Теперь создадим директории для файловой системы LFS. Для этого запускаем скрипт dir.sh
```
#!/bin/bash
mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}
for i in bin lib sbin; do
ln -sv usr/$i $LFS/$i
done
case $(uname -m) in
x86_64) mkdir -pv $LFS/lib64 ;;
esac
```
Дополнительно создаем директорию tools
```
mkdir -pv $LFS/tools
```
Структура $LFS будет выглядеть так:
```
/mnt/lfs
├── bin -> usr/bin
├── etc
├── lib -> usr/lib
├── lib64
├── local_files
├── lost+found [error opening dir]
├── sbin -> usr/sbin
├── sources
├── tools
├── usr
│   ├── bin
│   ├── lib
│   └── sbin
├── var
```
🔷 8. Подгототавливаем пользователя. Для чего это нужно создать пользователя с нуля. Зачем?
При добавлении пользователя в окружении к нему могут подтянуться общие настройки из определенных "скелетных" файлов, в том числе .bashrc, .profile и переменные окружения.  
У нас есть два файла, которые, в частности, отвечают за инициализацию переменных - .bashrc и .profile (bash profile)  
.profile подгружает наши переменные в том случае, если мы заходим удаленно на сервер. Когда мы вводим логин и пароль, в первую очередь стартует bash profile.  
Так же есть файл /etc/bash.bashrc, который тоже запускается при запуске командной оболочки, он нам для проекта LFS не нужен.  
Нам нужен чистый пользователь, у которго нет переменных, инициализированных в общей системе.  
Добавляем пользователя lfs, добавляем его в группу lfs, создаем для него пароль, даем ему права sudo и делаем его владельцем директории $LFS
```
groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
passwd lfs
usermod -aG sudo lfs
chown -v lfs $LFS/sources
```
---
В команде useradd:   
**-s /bin/bash** - устанавливаем оболочку по умолчанию  
**-g** - указываем группу пользователя  
**-m** - создаем домашнюю директорию  
**-k** - указание на скелетный файл, который должен быть основой для создания пользователя, в нашем случае ткого файла нет (/dev/null), т.е. домашний каталог пользователя будет пустой  

---

🔷 9. Настраиваем правильное рабочее окружение пользователя lfs.
В .bash_profile вставляем следующую строку
```
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
```
В этом файле устанавливаются значения переменных HOME, TERM, PS1, а также подтягивается файл bashrc.  
При входе в всистему пользователем lfs обычно считывается логин файл /etc/profile, содержащий несколько настроек и переменных окружения, а затем .bash_profile. Приведенная выше команда заменяет запускаемую по умолчанию оболочку новой с пустым окружением, за исключением переменных HOME, TERM и PS1.  
Новый экземпляр shell является non-login shell, который читает и исполняет содержимое файла .bashrc, а не файлов /etc/profile или bash profile.  
Устанавливаем следующее содержимое файла .bashrc
```
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
```
---
**set +h** - отключение hash-функции bash  
**umask** - устанавливаем права по умолчанию  
**LFS, LC_ALL, LFS_TGT** - переменные окруженияЮ, необходимые для сборки и компиляции  
**PATH** - путь до папки с исполняемыми файлами  
**if [ ! -L /bin ]; then PATH=/bin:$PATH; fi** - если в нашей системе каталог /bin является симлинком, то в переменную PATH добавляем каталог /bin  
**PATH=$LFS/tools/bin:$PATH** - добавляем в PATH $LFS/tools/bin  
**export LFS LC_ALL LFS_TGT PATH CONFIG_SITE** - экспортируем переменные, т.е. они будут доступны в дочерних командных оболочках  

---

🔷 10. Устанавливаем правильные права на домашний каталог lfs:
```
chown -R lfs:lfs /home/lfs
```

🔷 11. Если в системе есть файл /etc/bash.bashrc, то его необходимо убрать:
```
[ ! -e /etc/bash.bashrc ] || mv -v /etc/bash.bashrc /etc/bash.bashrc.NOUSE
```
   
🔷 12. И создаем профиль пользователя, предварительно залогинившись от его имени
```
source ~/.bash_profile
```

🔷 13. Устанавливаем права на каталог /mnt/lfs
```
chown -R lfs:lfs /mnt/lfs
```
🔷 14.  Устанавливаем дополнительные права на папку sources 1777 (стики бит - все пользователи хоста будут иметь право создавать файлы в каталоге sources (но не удалять))
```
chmod -v a+wt $LFS/sources
```
[Содержание](#000)

---

##  <a name='03'>PART 03</a> 
  
SBU - Standrad Build Unit, своего рода измерение времени процесса компиляциитой или иной программы. За оценочный параметр берется время компиляции пакета binutils. Именно с него и начинаем компиляцию.  

 ### 1. Chapter 5. Compiling a Cross-Toolchain  
 🔷 **Binutils-2.37**  
Распаковываем архив в папке sources
```
tar xvf binutils-2.38.tar.xz
````
Переходим в каталог /mnt/lfs/sources/binutils-2.38  
Применяем патч для пакета
```
patch -Np1 < ../binutils-2.38-lto_fix-1.patch
```
-N - позволяет продолжить применение патча, если произошла какая-то ошибка, с того же места  
p1 - убрать часть пути от исходного файла  

По окончании процесса проверяем код возврата, чтобы убедится, что не произошло ошибок
  ```
  lfs:/mnt/lfs/sources/binutils-2.38$ echo $?  
  0
  ```
Создаем директорию build и переходим в нее
```
mkdir -v build
cd build
```
Запускаем configure для создания Makefile
```
../configure --prefix=$LFS/tools --with-sysroot=$LFS --target=$LFS_TGT --disable-nls --disable-werror
```
--prefix - куда будет установлен пакет  
--with-sysroot - sysroot для пакета будет /mnt/lfs  
--disable-werror - отключаем предупреждения  

Проверяем код возврата и наличие Makefile. Mekefile - информация для декларативного языка make, инструкции, что нужно делать утилите make. Т.е. make не компилирует, а просто выполняет команды из этого файла, не более того.  
Запускаем make и измеряем время его работы. Используем 8 вычислительных потоков для нашей машины
```
time make -j8  
............................  
real    0m23.197s  
user    1m27.901s  
sys     0m11.603s  
lfs:/mnt/lfs/sources/binutils-2.38/build$ echo $?  
0
```
SBU в нашем случае = 0m23.197s  

Устанавливаем пакет
```
make install -j1  
lfs:/mnt/lfs/sources/binutils-2.38/build$ echo $?  
0
```
---
🔷 **GCC-11.2.0 - Pass 1**  
Распаковываем архив в папке sources
```
tar xvf gcc-11.2.0.tar.xz
````
Переходим в каталог /mnt/lfs/sources/gcc-11.2.0  
Устанавливаем дополнительные пакеты  
```
tar xvf ../mpfr-4.1.0.tar.xz  
mv -v mpfr-4.1.0 mpfr  
tar xvf ../gmp-6.2.1.tar.xz  
mv -v gmp-6.2.1 gmp  
tar xvf ../mpc-1.2.1.tar.gz  
mv -v mpc-1.2.1 mpc
```
Устанавливаем по умолчанию имя директории для 64-битных библиотек в "lib" через скрипт tmp.sh
```
case $(uname -m) in  
  x86_64)  
  sed -e '/m64=/s/lib64/lib/' \  
  -i.orig gcc/config/i386/t-linux64  
  ;;  
esac
```
Создаем директорию build и переходим в нее
```
mkdir -v build
cd build
```
Запускаем configure
```
../configure --target=$LFS_TGT --prefix=$LFS/tools --with-glibc-version=2.11 --with-sysroot=$LFS --with-newlib --without-headers --enable-initfini-array --disable-nls --disable-shared  -disable-multilib --disable-decimal-float --disable-threads --disable-libatomic --disable-libgomp  --disable-libquadmath --disable-libssp --disable-libvtv --disable-libstdcxx --enable-languages=c,c++
```
Запускаем make
```
time make -j8  
............................ 
real    2m57.347s  
user    15m55.439s  
sys     1m16.443s
```
Устанавливаем пакет
```
make install   
lfs:/mnt/lfs/sources/gcc-11.2.0/build$ echo $?  
0
```
Порпавляем заголовочные файлы 
```
cd ..  
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > `dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/install-tools/include/limits.h  
```
---
🔷 **Linux-5.13.12 API Headers**  
Распаковываем ядро и переходим в его каталог
```
lfs:/mnt/lfs/sources$ tar xvf linux-5.16.9.tar.xz
```
Зачищаем то, что нам не нужно
```
make mrproper
```
Извлекаем доступные пользователю заголовки ядра, копируем их в нужное расположение
```
make headers  
find usr/include -name '.*' -delete  
rm usr/include/Makefile  
cp -rv usr/include $LFS/usr
```
Хэдеры готовы.

🔷 **Glibc-2.35**  
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf glibc-2.35.tar.xz
```
Создаем необходимые символические ссылки через запуск файла tmp.sh
```
case $(uname -m) in
  i?86) ln -sfv ld-linux.so.2 $LFS/lib/ld-lsb.so.3
  ;;
  x86_64) ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64
  ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64/ld-lsb-x86-64.so.3
  ;;
esac  
```
Устанавливаем патч
```
patch -Np1 < ../glibc-2.35-fhs-1.patch
```
Создаем директорию build и переходим в нее
```
mkdir -v build
cd build
```
Выполняем команду, которая задает параметры конфигурации
```
echo "rootsbindir=/usr/sbin" > configparms
```
Конфигурируем make
```
../configure --prefix=/usr --host=$LFS_TGT --build=$(../scripts/config.guess) --enable-kernel=3.2 --with-headers=$LFS/usr/include libc_cv_slibdir=/usr/lib
```
Выполняем make
```
time make -j8
.......................................
real    1m14.629s  
user    6m8.734s  
sys     1m22.353s  
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install   
lfs:/mnt/lfs/sources/glibc-2.35/build$ echo $?  
0
```
Переменная DESTDIR используется поти всеми пакетами для определения места,Ю куда пакеты будут утсановлены. Если она не определена, то ее значение равно / (root директория).  
Корректируем файл ldd, который описывает библиотеки
```
sed '/RTLDLIST=/s@/usr@@g' -i $LFS/usr/bin/ldd
```
Проверяем, работает ли это все
```
echo 'int main(){}' > dummy.c  
$LFS_TGT-gcc dummy.c  
readelf -l a.out | grep '/ld-linux'  
    [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```
Последнне сообщение говорит о том, что все работает корректно.  
Удаляем тестовые файлы
```
rm -v dummy.c a.out
```
В финале создаем хэдер limits.h
```
$LFS/tools/libexec/gcc/$LFS_TGT/11.2.0/install-tools/mkheaders
```
Удаляем каталог build
```
lfs:/mnt/lfs/sources/glibc-2.35$ rm -rf build/
```
---
🔷 **GCC-11.2.0, Pass 1**  
Если каталог /mnt/lfs/sources/gcc-11.2.0 не удалялся, то переходим в него и удаляем каталог build, а затем его заново создаем. Если каталог удалялся, то заново распаковываем архив и создаем в каталоге gcc-11.2.0 папку build
```
mnt/lfs/sources$ tar xvf gcc-11.2.0.tar.xz  
cd gcc-11.2.0
mkdir -v build
cd build
```
Подготавливаем libstdc++ для компиляции
```
../libstdc++-v3/configure --host=$LFS_TGT --build=$(../config.guess) --prefix=/usr --disable-multilib --disable-nls --disable-libstdcxx-pch --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/11.2.0
```
Выполняем make
```
make -j8
```
Устанавливаем библиотеку с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install
```
