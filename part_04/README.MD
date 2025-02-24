# Часть 4

## 6. Cross Compiling Temporary Tools

### 🔷 M4-1.4.19

Распаковываем исходники пакета и переходим в его каталог
```
tar xvf m4-1.4.19.tar.xz  
cd cd m4-1.4.19
```
Конфигурируем make
```
./configure --prefix=/usr --host=$LFS_TGT --build=$(build-aux/config.guess)
```
Комилируем пакет
```
time make -j8
.......................................
real    0m2.990s  
user    0m9.277s  
sys     0m1.334s  
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install   
lfs:/mnt/lfs/sources/m4-1.4.19$ echo $?
0
```
Удаляем ненужные файлы
```
cd ..  
rm -rf m4-1.4.19
```
---
### 🔷 Ncurses-6.3
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf ncurses-6.3.tar.gz  
cd ncurses-6.3
```
Убираем из скрипта configure все упоминания об утилите mawk
```
sed -i s/mawk// configure
```
Создаем директорию build
```
mkdir -v build
```
Создаем стэк каталогов
```
pushd build
/mnt/lfs/sources/ncurses-6.3/build /mnt/lfs/sources/ncurses-6.3
```
Т.е. теперь при запуске команды popd мы перейдем из одного каталога /mnt/lfs/sources/ncurses-6.3/build в каталог/mnt/lfs/sources/ncurses-6.3  
Запускаем команды для сборки программы tic и переходим в вышестоящий каталог
```
../configure  
make -C include  
make -C progs tic  
popd  
```
Готовим ncurces для компиляции
```
./configure --prefix=/usr --host=$LFS_TGT --build=$(./config.guess) --mandir=/usr/share/man --with-manpage-format=normal --with-shared --without-debug --without-ada --without-normal --enable-widec
```
Комилируем пакет
```
time make -j8
.......................................
real    0m5.661s  
user    0m25.875s  
sys     0m4.104s  
lfs:/mnt/lfs/sources/ncurses-6.3$ echo $?  
0
```
Устанавливаем пакет с инициализацией переменных
```
make DESTDIR=$LFS TIC_PATH=$(pwd)/build/progs/tic install
lfs:/mnt/lfs/sources/ncurses-6.3$ echo $?
0
```
Записываем параметр "INPUT(-lncursesw)" в библиотеку libncurses.so
```
echo "INPUT(-lncursesw)" > $LFS/usr/lib/libncurses.so
```
Удаляем ненужные файлы
```
cd ..  
rm -rf ncurses-6.3

```
---

### 🔷 Bash-5.1.16
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf bash-5.1.16.tar.gz  
cd bash-5.1.16
```
Готовим bash для компиляции
```
./configure --prefix=/usr --build=$(support/config.guess) --host=$LFS_TGT --without-bash-malloc
```
Комилируем пакет
```
time make -j8
.......................................
real    0m3.735s  
user    0m21.924s  
sys     0m2.107s  
lfs:/mnt/lfs/sources/bash-5.1.16$ echo $?  
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/bash-5.1.16$ echo $?  
0
```
Создаем ссылку для программ, использующих sh в качестве оболочки
```
ln -sv bash $LFS/bin/sh
```
Удаляем ненужные файлы
```
cd ..  
rm -rf bash-5.1.16
```
---
### 🔷 Coreutils-9.0
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf coreutils-9.0.tar.xz   
cd coreutils-9.0
```
Готовим Coreutils для компиляции
```
./configure --prefix=/usr --host=$LFS_TGT --build=$(build-aux/config.guess) --enable-install-program=hostname --enable-no-install-program=kill,uptime
```
Компилируем пакет
```
time make -j8
.......................................
real    0m6.064s  
user    0m38.977s  
sys     0m5.021s  
lfs:/mnt/lfs/sources/coreutils-9.0$ echo $?  
0  
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/coreutils-9.0$ echo $?  
0
```
Создаем страницы руководства, чтобы программы не выдавали ошибок об их отсутствии
```
mv -v $LFS/usr/bin/chroot $LFS/usr/sbin  
mkdir -pv $LFS/usr/share/man/man8  
mv -v $LFS/usr/share/man/man1/chroot.1 $LFS/usr/share/man/man8/chroot.8  
sed -i 's/"1"/"8"/' $LFS/usr/share/man/man8/chroot.8  
```
Удаляем ненужные файлы
```
cd ..  
rm -rf coreutils-9.0
```
---

### 🔷 Diffutils-3.8
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf diffutils-3.8.tar.xz     
cd diffutils-3.8
```
Готовим Diffutils для компиляции
```
./configure --prefix=/usr --host=$LFS_TGT
```
Компилируем пакет
```
time make -j8  
.......................................  
real    0m1.295s  
user    0m6.030s  
sys     0m0.956s  
lfs:/mnt/lfs/sources/diffutils-3.8$ echo $?  
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/diffutils-3.8$ echo $?
0
```
Удаляем ненужные файлы
```
cd ..  
rm -rf diffutils-3.8
```
---
### 🔷 File-5.41
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf file-5.41.tar.gz       
cd file-5.41
```
Создаем директорию build
```
mkdir -v build
```
Собираем файл сигнатуры
```
pushd build  
../configure --disable-bzlib --disable-libseccomp --disable-xzlib --disable-zlib  
make  
popd  
```
Готовим File для компиляции
```
./configure --prefix=/usr --host=$LFS_TGT --build=$(./config.guess)
```
Компилируем пакет
```
time make -j8 ILE_COMPILE=$(pwd)/build/src/file  
.......................................  
real    0m1.037s  
user    0m4.348s  
sys     0m0.551s  
lfs:/mnt/lfs/sources/file-5.41$ echo $?  
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/file-5.41$ echo $?  
0
```
Удаляем ненужные файлы
```
cd ..  
rm -rf file-5.41
```
---

### 🔷 Findutils-4.9.0
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf findutils-4.9.0.tar.xz && cd findutils-4.9.0
```
Подготавливаем Findutils для компиляции
```
./configure --prefix=/usr --localstatedir=/var/lib/locate --host=$LFS_TGT --build=$(build-aux/config.guess)
```
Компилируем пакет
```
time make -j8
.......................................  
real    0m3.320s
user    0m10.131s
sys     0m1.506s
lfs:/mnt/lfs/sources/findutils-4.9.0$ echo $?
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/findutils-4.9.0$ echo $?
0
```
Удаляем ненужные файлы
```
cd ..  
rm -rf findutils-4.9.0
```
---

### 🔷 Gawk-5.1.1
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf gawk-5.1.1.tar.xz && cd gawk-5.1.1
```
Убеждаемся, что не нужные файлы не установлены
```
sed -i 's/extras//' Makefile.in
```
Подготавливаем Gawk для компиляции
```
./configure --prefix=/usr --host=$LFS_TGT --build=$(./config.guess)
```
Компилируем пакет
```
time make -j8  
.......................................  
real    0m3.393s
user    0m14.066s
sys     0m0.995s
lfs:/mnt/lfs/sources/gawk-5.1.1$ echo $?
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/gawk-5.1.1$ echo $?
0
```
Удаляем ненужные файлы
```
cd ..  && rm -rf gawk-5.1.1
```
---

### 🔷 Grep-3.7
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf grep-3.7.tar.xz        
cd grep-3.7
```
Подготавливаем Grep для компиляции
```
./configure --prefix=/usr --host=$LFS_TGT
```
Компилируем пакет
```
time make -j8  
.......................................  
real    0m1.597s
user    0m5.722s
sys     0m0.892s
lfs:/mnt/lfs/sources/grep-3.7$ echo $?
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/grep-3.7$ echo $?
0
```
Удаляем ненужные файлы
```
cd ..  
rm -rf grep-3.7
```
---

### 🔷 Gzip-1.11
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf gzip-1.11.tar.xz         
cd gzip-1.11  
```
Подготавливаем Gzip для компиляции
```
./configure --prefix=/usr --host=$LFS_TGT
```
Компилируем пакет
```
time make -j8  
.......................................  
real    0m0.797s
user    0m2.981s
sys     0m0.583s
lfs:/mnt/lfs/sources/gzip-1.11$ echo $?
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/gzip-1.11$ echo $?  
0
```
Удаляем ненужные файлы
```
cd ..  
rm -rf gzip-1.11
```
---

### 🔷 Make-4.3
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf make-4.3.tar.gz           
cd make-4.3  
```
Подготавливаем Make для компиляции
```
./configure --prefix=/usr --without-guile --host=$LFS_TGT --build=$(build-aux/config.guess)
```
Компилируем пакет
```
time make -j8  
.......................................  
real    0m0.819s
user    0m4.098s
sys     0m0.384s
lfs:/mnt/lfs/sources/make-4.3$ echo $?
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/make-4.3$ echo $?  
0
```
Удаляем ненужные файлы
```
cd ..  
rm -rf make-4.3
```
---
### 🔷 Patch-2.7.6
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf patch-2.7.6.tar.xz           
cd patch-2.7.6  
```
Подготавливаем Patch для компиляции
```
./configure --prefix=/usr --host=$LFS_TGT --build=$(build-aux/config.guess)
```
Компилируем пакет
```
time make -j8  
.......................................  
real    0m1.166s
user    0m4.847s
sys     0m0.512s
lfs:/mnt/lfs/sources/patch-2.7.6$ echo $?
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/patch-2.7.6$ echo $?  
0
```
Удаляем ненужные файлы
```
cd ..  
rm -rf patch-2.7.6
```
---

### 🔷 Sed-4.8
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf sed-4.8.tar.xz           
cd sed-4.8  
```
Подготавливаем Sed для компиляции
```
./configure --prefix=/usr --host=$LFS_TGT
```
Компилируем пакет
```
time make -j8  
.......................................  
real    0m2.038s
user    0m5.333s
sys     0m0.561s
lfs:/mnt/lfs/sources/sed-4.8$ echo $?
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/sed-4.8$ echo $?  
0
```
Удаляем ненужные файлы
```
cd ..  
rm -rf sed-4.8
```
---

### 🔷 Tar-1.34
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf tar-1.34.tar.xz            
cd tar-1.34  
```
Подготавливаем Tar для компиляции
```
./configure --prefix=/usr --host=$LFS_TGT --build=$(build-aux/config.guess)
```
Компилируем пакет
```
time make -j8  
.......................................  
real    0m3.752s
user    0m13.185s
sys     0m1.423s
lfs:/mnt/lfs/sources/tar-1.34$ echo $?
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/tar-1.34$ echo $?  
0
```
Удаляем ненужные файлы
```
cd ..  
rm -rf tar-1.34
```
---

### 🔷 Xz-5.2.5
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf xz-5.2.5.tar.xz            
cd xz-5.2.5  
```
Подготавливаем Xz для компиляции
```
./configure --prefix=/usr --host=$LFS_TGT --build=$(build-aux/config.guess) --disable-static --docdir=/usr/share/doc/xz-5.2.5
```
Компилируем пакет
```
time make -j8  
.......................................  
real    0m1.962s
user    0m8.483s
sys     0m1.598s
lfs:/mnt/lfs/sources/xz-5.2.5$ echo $?
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install  
lfs:/mnt/lfs/sources/xz-5.2.5$ echo $?  
0
```
Удаляем ненужные файлы
```
cd ..  
rm -rf xz-5.2.5
```
---

### 🔷 Binutils-2.38 - Pass 2
Распаковываем исходники пакета, переходим в его каталог и устанавливаем патч
```
tar xvf binutils-2.38.tar.xz  
cd binutils-2.38
patch -Np1 < ../binutils-2.38-lto_fix-1.patch
```
Уберем из 6009 строки add_dir в файле ltmain.sh
```
sed '6009s/$add_dir//' -i ltmain.sh
```
Создаем директорию build и переходим в нее
```
mkdir -v build
cd build
```
Подготавливаем Binutils для компиляции
```
../configure --prefix=/usr --build=$(../config.guess) --host=$LFS_TGT --disable-nls --enable-shared --disable-werror --enable-64-bit-bfd
```
---
--enable-shared - собираем libbfd как разделяемую библиотеку
--enable-64-bit-bfd - активируем поддержку 64 бит
---
Компилируем пакет
```
time make -j8  
.......................................  
real    0m28.733s
user    2m0.590s
sys     0m13.585s
lfs:/mnt/lfs/sources/binutils-2.38/build$ echo $?
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install -j1 
lfs:/mnt/lfs/sources/binutils-2.38/build$ echo $? 
0
```
Устанавливаем рабочее окружение libctf.so
```
install -vm755 libctf/.libs/libctf.so.0.0.0 $LFS/usr/lib
```
Удаляем ненужные файлы
```
cd ..  
cd ..
rm -rf binutils-2.38
```
---

### 🔷 GCC-11.2.0 - Pass 2
Распаковываем архив в папке sources и переходим в папку gcc
```
tar xvf gcc-11.2.0.tar.xz
cd gcc-11.2.0
````
Устанавливаем дополнительные пакеты  
```
tar -xf ../mpfr-4.1.0.tar.xz
mv -v mpfr-4.1.0 mpfr
tar -xf ../gmp-6.2.1.tar.xz
mv -v gmp-6.2.1 gmp
tar -xf ../mpc-1.2.1.tar.gz
mv -v mpc-1.2.1 mpc
```
Устанавливаем по умолчанию имя директории для 64-битных библиотек в "lib" через скрипт tmp.sh
```
case $(uname -m) in
   x86_64)
     sed -e '/m64=/s/lib64/lib/' -i.orig gcc/config/i386/t-linux64
   ;;
esac
```
Создаем директорию build и переходим в нее
```
mkdir -v build
cd build
```
Создаем симлинки, которые позволяют собрать libgcc с поддержкой потоков posix
```
mkdir -pv $LFS_TGT/libgcc  
ln -s ../../../libgcc/gthr-posix.h $LFS_TGT/libgcc/gthr-default.h
```
Подготавливаем GCC для компиляции
```
../configure --build=$(../config.guess) --host=$LFS_TGT --prefix=/usr CC_FOR_TARGET=$LFS_TGT-gcc --with-build-sysroot=$LFS --enable-initfini-array --disable-nls --disable-multilib --disable-decimal-float --disable-libatomic --disable-libgomp --disable-libquadmath --disable-libssp --disable-libvtv --disable-libstdcxx --enable-languages=c,c++
```
Компилируем пакет
```
time make -j8  
.......................................  
real    3m15.829s
user    18m0.297s
sys     1m20.780s
lfs:/mnt/lfs/sources/gcc-11.2.0/build$ echo $?
0
```
Устанавливаем пакет с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install 
lfs:/mnt/lfs/sources/gcc-11.2.0/build$ echo $?
0
```
Создаем симлинк
```
ln -sv gcc $LFS/usr/bin/cc
............................  
'/mnt/lfs/usr/bin/cc' -> 'gcc'
```
Удаляем ненужные файлы
```
cd ..  
cd ..
rm -rf gcc-11.2.0
```
---



