# Часть 7

### 🔷 Man-pages-5.13

Распаковываем Man-pages-5.13 и переходим в папку с пакетом
```
tar xvf man-pages-5.13.tar.xz && cd man-pages-5.13
```
Устанавливаем пакет
```
make prefix=/usr install
```
```
(lfs chroot) root:/sources/man-pages-5.13# echo $?
0
```
Удаляем ненужные файлы
```
cd .. && rm -rf man-pages-5.13
```
---

### 🔷 Iana-Etc-20220207

Распаковываем исходники и переходим в папку с пакетом
```
tar xvf iana-etc-20220207.tar.gz && cd iana-etc-20220207
```
Устанавливаем пакет
```
cp services protocols /etc
```
Удаляем ненужные файлы
```
cd .. && rm -rf iana-etc-20220207
```
---

### 🔷 Glibc-2.35

Распаковываем исходники и переходим в папку с пакетом
```
tar xvf glibc-2.35.tar.xz && cd glibc-2.35
```
Применяем патч
```
patch -Np1 -i ../glibc-2.35-fhs-1.patch
```
Создаем директорию build и переходим в нее
```
mkdir -v build && cd build
```
Убеждаемся что утилиты ldconfig и sln будут установлены в /usr/sbin:
```
echo "rootsbindir=/usr/sbin" > configparms
```
Готовим Glibc для компиляции
```
../configure --prefix=/usr --disable-werror --enable-kernel=3.2 --enable-stack-protector=strong --with-headers=/usr/include libc_cv_slibdir=/usr/lib
```
Компилируем пакет
```
time make -j8
```
```
real    1m14.789s
user    6m5.669s
sys     1m23.272s
(lfs chroot) root:/sources/glibc-2.35/build# echo $?
0
```
Запускаем тесты
```
time make -j8 check
```
```
Summary of test results:
      2 FAIL
   4884 PASS
    227 UNSUPPORTED
     16 XFAIL
      6 XPASS
make[1]: *** [Makefile:646: tests] Error 1
make[1]: Leaving directory '/sources/glibc-2.35'
make: *** [Makefile:9: check] Error 2

real    14m36.704s
user    30m13.523s
sys     7m31.688s
```
Создаем /etc/ld.so.conf
```
touch /etc/ld.so.conf
```
Убираем из Makefile не нужные проверки
```
sed '/test-installation/s@$(PERL)@echo not running@' -i ../Makefile
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/glibc-2.35/build# echo $?
0
```
Исправляем жестко запрограммированный путь к исполняемому загрузчику в скрипте ldd
```
sed '/RTLDLIST=/s@/usr@@g' -i /usr/bin/ldd
```
Устанавливаем конфигурационный файл и исполняемую директорию для nscd
```
cp -v ../nscd/nscd.conf /etc/nscd.conf && mkdir -pv /var/cache/nscd
```
Устанавливаем региональные настройки 
```
mkdir -pv /usr/lib/locale
localedef -i POSIX -f UTF-8 C.UTF-8 2> /dev/null || true
localedef -i cs_CZ -f UTF-8 cs_CZ.UTF-8
localedef -i de_DE -f ISO-8859-1 de_DE
localedef -i de_DE@euro -f ISO-8859-15 de_DE@euro
localedef -i de_DE -f UTF-8 de_DE.UTF-8
localedef -i el_GR -f ISO-8859-7 el_GR
localedef -i en_GB -f ISO-8859-1 en_GB
localedef -i en_GB -f UTF-8 en_GB.UTF-8
localedef -i en_HK -f ISO-8859-1 en_HK
localedef -i en_PH -f ISO-8859-1 en_PH
localedef -i en_US -f ISO-8859-1 en_US
localedef -i en_US -f UTF-8 en_US.UTF-8
localedef -i es_ES -f ISO-8859-15 es_ES@euro
localedef -i es_MX -f ISO-8859-1 es_MX
localedef -i fa_IR -f UTF-8 fa_IR
localedef -i fr_FR -f ISO-8859-1 fr_FR
localedef -i fr_FR@euro -f ISO-8859-15 fr_FR@euro
localedef -i fr_FR -f UTF-8 fr_FR.UTF-8
localedef -i is_IS -f ISO-8859-1 is_IS
localedef -i is_IS -f UTF-8 is_IS.UTF-8
localedef -i it_IT -f ISO-8859-1 it_IT
localedef -i it_IT -f ISO-8859-15 it_IT@euro
localedef -i it_IT -f UTF-8 it_IT.UTF-8
localedef -i ja_JP -f EUC-JP ja_JP
localedef -i ja_JP -f SHIFT_JIS ja_JP.SJIS 2> /dev/null || true
localedef -i ja_JP -f UTF-8 ja_JP.UTF-8
localedef -i nl_NL@euro -f ISO-8859-15 nl_NL@euro
localedef -i ru_RU -f KOI8-R ru_RU.KOI8-R
localedef -i ru_RU -f UTF-8 ru_RU.UTF-8
localedef -i se_NO -f UTF-8 se_NO.UTF-8
localedef -i ta_IN -f UTF-8 ta_IN.UTF-8
localedef -i tr_TR -f UTF-8 tr_TR.UTF-8
localedef -i zh_CN -f GB18030 zh_CN.GB18030
localedef -i zh_HK -f BIG5-HKSCS zh_HK.BIG5-HKSCS
localedef -i zh_TW -f UTF-8 zh_TW.UTF-8
```
Также можно добавить настройки для своего региона или все региональные настройки из файла glibc-2.35/localedata/SUPPORTED
```
make localedata/install-locales
```
Затем введем команду localedef для использования настроек регионов, не входящих в список glibc-2.35/localedata/SUPPORTED. Например, следующие две локали нужны для некоторых тестов
```
localedef -i POSIX -f UTF-8 C.UTF-8 2> /dev/null || true  
localedef -i ja_JP -f SHIFT_JIS ja_JP.SJIS 2> /dev/null || true
```
---

### 🔷 Конфигурирование Glibc
#### Добавляем nsswitch.conf
```
cat > /etc/nsswitch.conf << "EOF"
# Begin /etc/nsswitch.conf

passwd: files
group: files
shadow: files

hosts: files dns
networks: files

protocols: files
services: files
ethers: files
rpc: files

# End /etc/nsswitch.conf
EOF
```
#### Добавляем time zone data
Устанавливаем и настраиваем time zone data следующим образом
```
tar -xf ../../tzdata2021e.tar.gz

ZONEINFO=/usr/share/zoneinfo
mkdir -pv $ZONEINFO/{posix,right}

for tz in etcetera southamerica northamerica europe africa antarctica  \
          asia australasia backward; do
    zic -L /dev/null   -d $ZONEINFO       ${tz}
    zic -L /dev/null   -d $ZONEINFO/posix ${tz}
    zic -L leapseconds -d $ZONEINFO/right ${tz}
done

cp -v zone.tab zone1970.tab iso3166.tab $ZONEINFO
zic -d $ZONEINFO -p America/New_York
unset ZONEINFO
```
Определяем нужную нам временную зону
```
tzselect
```
Для нашего региона была выбрана зона Europe/Moscow.  
Далее создаем файл /etc/localtime
```
ln -sfv /usr/share/zoneinfo/Europe/Moscow /etc/localtime
```
#### Конфигурируем Dynamic Loader
Настраиваем файл, отвечающий за информацию, где хранятся наши библиотеки.  
Создаем новый файл /etc/ld.so.conf 
```
cat > /etc/ld.so.conf << "EOF"
# Begin /etc/ld.so.conf
/usr/local/lib
/opt/lib

EOF
```
Если мы хотим формировать собственные дополнительные файлы, то мы можем прописывать путь до них в /etc/ld.so.conf.d в файл *.conf. Для этого прописываем директорию
```
cat >> /etc/ld.so.conf << "EOF"
# Add an include directory
include /etc/ld.so.conf.d/*.conf

EOF
mkdir -pv /etc/ld.so.conf.d
```
---

### 🔷 Zlib-1.3.1

Распаковываем исходники и переходим в папку с пакетом
```
tar xvf zlib-1.3.1.tar.xz && cd zlib-1.3.1
```
Готовим Zlib для компиляции
```
./configure --prefix=/usr
```
Компилируем пакет
```
time make -j8
```
```
real    0m0.621s
user    0m3.789s
sys     0m0.298s
(lfs chroot) root:/sources/zlib-1.3.1# echo $?
0
```
Запускаем тесты
```
time make -j8 check
```
```
.............................
*** zlib test OK ***
.............................
*** zlib shared test OK ***

real    0m0.007s
user    0m0.013s
sys     0m0.002s
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/zlib-1.3.1# echo $?
0
```
Удаляем ненужную статическую библиотеку
```
rm -fv /usr/lib/libz.a
```
Удаляем исходные файлы из source
```
cd .. && rm -rf zlib-1.3.1
```
---











