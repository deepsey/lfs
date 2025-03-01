# Часть 6

### 🔷 Libstdc++ from GCC-11.2.0, Pass 2

Libstdc++ является частью GCC. Поэтому сначала распаковываем gcc 11.0.2
```
cd sources
```
```
tar xvf gcc-11.2.0.tar.xz && cd gcc-11.2.0 
```
Линкуем хэдер в нужное место
```
ln -s gthr-posix.h libgcc/gthr-default.h
```
Создаем директорию build и переходим в нее
```
mkdir -v build && cd build
```
Готовим libstdc++ для компиляции
```
../libstdc++-v3/configure CXXFLAGS="-g -O2 -D_GNU_SOURCE" --prefix=/usr --disable-multilib --disable-nls --host=$(uname -m)-lfs-linux-gnu --disable-libstdcxx-pch
```
Компилируем пакет
```
time make -j8
```
```
real    0m14.963s
user    1m2.407s
sys     0m4.954s
(lfs chroot) root:/sources/gcc-11.2.0/build# echo $?
0
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/gcc-11.2.0/build# echo $?
0
```
Удаляем ненужные файлы
```
cd .. && cd .. && rm -rf gcc-11.2.0
```
---

### 🔷 Gettext-0.21
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf gettext-0.21.tar.xz && cd gettext-0.21
```
Готовим Gettext для компиляции
```
./configure --disable-shared
```
disable-shared - нам пока не нужно устанавливать разделяемые библиотеки пакета
Компилируем пакет
```
time make -j8
```
```
real    1m0.079s
user    1m47.835s
sys     0m12.578s
(lfs chroot) root:/sources/gettext-0.21# echo $?
0
```
Устанавливаем the msgfmt, msgmerge, и xgettext
```
cp -v gettext-tools/src/{msgfmt,msgmerge,xgettext} /usr/bin
```
Удаляем ненужные файлы
```
cd .. && rm -rf gettext-0.21
```
---

### 🔷 Bison-3.8.2
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf bison-3.8.2.tar.xz && cd bison-3.8.2
```
Готовим Bison для компиляции
```
./configure --prefix=/usr --docdir=/usr/share/doc/bison-3.8.2
```
Компилируем пакет
```
time make -j8
```
```
real    0m1.975s
user    0m12.277s
sys     0m1.157s
(lfs chroot) root:/sources/bison-3.8.2# echo $?
0
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/bison-3.8.2# echo $?
0
```
Удаляем ненужные файлы
```
cd .. && rm -rf bison-3.8.2
```
---

### 🔷 Perl-5.34.0 
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf perl-5.34.0.tar.xz && cd perl-5.34.0
```
Готовим Perl для компиляции
```
sh Configure -des -Dprefix=/usr -Dvendorprefix=/usr -Dprivlib=/usr/lib/perl5/5.34/core_perl -Darchlib=/usr/lib/perl5/5.34/core_perl -Dsitelib=/usr/lib/perl5/5.34/site_perl -Dsitearch=/usr/lib/perl5/5.34/site_perl -Dvendorlib=/usr/lib/perl5/5.34/vendor_perl -Dvendorarch=/usr/lib/perl5/5.34/vendor_perl
```
des - комбинация их трех опций: -d - использовать параметры по умолчанию (defaults) для всех компонентов, -e (ensure) - убедиться в выполеннии всех задач, -s (silences) - не выводить несущественные сообщения  
Компилируем пакет
```
time make -j8
```
```
real    0m22.391s
user    1m58.395s
sys     0m8.059s
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/perl-5.34.0# echo $?
0
```
Удаляем ненужные файлы
```
cd .. && rm -rf perl-5.34.0
```
---

### 🔷 Python-3.10.2
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf Python-3.10.2.tar.xz && cd Python-3.10.2
```
Готовим Python для компиляции
```
./configure --prefix=/usr --enable-shared --without-ensurepip
```
Компилируем пакет
```
time make -j8
```
```
real    0m19.232s
user    1m36.391s
sys     0m5.490s
(lfs chroot) root:/sources/Python-3.10.2# echo $?
0
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/Python-3.10.2# echo $?
0
```
Удаляем ненужные файлы
```
cd .. && rm -rf Python-3.10.2
```
---

### 🔷 Texinfo-6.8
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf texinfo-6.8.tar.xz && cd texinfo-6.8
```
Сначала исправляем проблемы сборки пакета с Glibc-2.34 или более поздних версий
```
sed -e 's/__attribute_nonnull__/__nonnull/' -i gnulib/lib/malloc/dynarray-skeleton.c
```
Готовим Texinfo для компиляции
```
./configure --prefix=/usr
```
Компилируем пакет
```
time make -j8
```
```
real    0m4.632s
user    0m10.540s
sys     0m1.219s
(lfs chroot) root:/sources/texinfo-6.8# echo $?
0
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/texinfo-6.8# echo $?
0
```
Удаляем ненужные файлы
```
cd .. && rm -rf texinfo-6.8  
```
---

### 🔷 Util-linux-2.37.4
Распаковываем исходники пакета и переходим в его каталог
```
tar xvf util-linux-2.37.4.tar.xz && cd util-linux-2.37.4
```
FHS рекомендует использовать директорию /var/lib/hwclock вместо /etc для размещения файла adjtime
```
mkdir -pv /var/lib/hwclock
```
Готовим Util-linux для компиляции
```
./configure ADJTIME_PATH=/var/lib/hwclock/adjtime --libdir=/usr/lib --docdir=/usr/share/doc/util-linux-2.37.2 --disable-chfn-chsh --disable-login --disable-nologin --disable-su --disable-setpriv --disable-runuser --disable-pylibmount --disable-static --without-python  runstatedir=/run
```
Компилируем пакет
```
time make -j8
```
```
real    0m9.171s
user    0m57.860s
sys     0m6.677s
(lfs chroot) root:/sources/util-linux-2.37.4# echo $?
0
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/util-linux-2.37.4# echo $?
0
```
Удаляем ненужные файлы
```
cd .. && rm -rf util-linux-2.37.4  
```
---

## 7.14. Cleaning up and Saving the Temporary System
### 🔷 Очищаем систему
Удаляем установленную документиацию
```
rm -rf /usr/share/{info,man,doc}/*
```
Удаляем файлы libtool .la
```
find /usr/{lib,libexec} -name \*.la -delete
```
Удаляем директорию /tools
```
rm -rf /tools
```
---

### 🔷 Делаем резервную копию
Для этого используем средства гипервизора, предварительно выключив VM

