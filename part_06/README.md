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




