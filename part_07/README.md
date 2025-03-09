# Часть 7

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
