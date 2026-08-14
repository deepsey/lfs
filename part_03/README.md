#  Часть 3 
  
SBU - Standrad Build Unit, своего рода измерение времени процесса компиляциитой или иной программы. За оценочный параметр берется время компиляции пакета binutils. Именно с него и начинаем компиляцию.  

 ## Chapter 5. Compiling a Cross-Toolchain  
### 🔷 Binutils-2.38  
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
### 🔷 GCC-11.2.0 - Pass 1  
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
  sed -e '/m64=/s/lib64/lib/' -i.orig gcc/config/i386/t-linux64
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
Поправляем заголовочные файлы 
```
cd ..  
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > `dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/install-tools/include/limits.h  
```
---
### 🔷 Linux-5.16.9 API Headers
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

---

### 🔷 Glibc-2.35
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
Переменная DESTDIR используется поти всеми пакетами для определения места, куда пакеты будут установлены. Если она не определена, то ее значение равно / (root директория).  
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

### 🔷 Libstdc++ from GCC-11.2.0, Pass 1
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
time make -j8
```
```
real    0m7.577s
user    0m30.290s
sys     0m3.524s
lfs:/mnt/lfs/sources/gcc-11.2.0/build$ echo $?
0
```
Устанавливаем GCC-11.2.0 с инициализацией переменной DESTDIR=$LFS
```
make DESTDIR=$LFS install
```
```
lfs:/mnt/lfs/sources/gcc-11.2.0/build$ echo $?
0
```
