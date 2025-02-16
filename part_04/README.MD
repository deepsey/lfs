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

