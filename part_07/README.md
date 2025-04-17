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
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf zlib-1.3.1
```
---

### 🔷 Bzip2-1.0.8

Распаковываем исходники и переходим в папку с пакетом
```
tar xvf bzip2-1.0.8.tar.gz && cd bzip2-1.0.8
```
Применяем патч с документаций для этого пакета
```
patch -Np1 -i ../bzip2-1.0.8-install_docs-1.patch
```
Выполняем команду для правильной установки симлинков
```
sed -i 's@\(ln -s -f \)$(PREFIX)/bin/@\1@' Makefile
```
Убеждаемся, что страницы руководства установлены в правильную локацию
```
sed -i "s@(PREFIX)/man@(PREFIX)/share/man@g" Makefile
```
Готовим Bzip2 для компиляции
```
make -f Makefile-libbz2_so
make clean
```
Компилируем пакет и тестируем пакет
```
time make -j8
```
```
real    0m0.818s
user    0m2.048s
sys     0m0.140s
(lfs chroot) root:/sources/bzip2-1.0.8# echo $?
0
```
Устанавливаем программы
```
make PREFIX=/usr install
```
```
(lfs chroot) root:/sources/bzip2-1.0.8# echo $?                 
0
```
Устанавливаем разделяемую библиотеку
```
cp -av libbz2.so.* /usr/lib
ln -sv libbz2.so.1.0.8 /usr/lib/libbz2.so
```
Устанавливаем разделеляемый бинарник bzip2 в директорию /usr/bin, и удаляем две копии bzip2 при помощи симлинков
```
cp -v bzip2-shared /usr/bin/bzip2
```
```
for i in /usr/bin/{bzcat,bunzip2}; do ln -sfv bzip2 $i; done
```
Удаляем ненужную статическую библиотеку
```
rm -fv /usr/lib/libbz2.a
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf bzip2-1.0.8
```
---

### 🔷 Xz-5.2.5

Распаковываем исходники и переходим в папку с пакетом
```
tar xvf xz-5.2.5.tar.xz && cd xz-5.2.5
```
Готовим Xz для компиляции
```
./configure --prefix=/usr --disable-static --docdir=/usr/share/doc/xz-5.2.5
```
Компилируем пакет
```
time make -j8
```
```
real    0m1.928s
user    0m8.224s
sys     0m1.527s
(lfs chroot) root:/sources/xz-5.2.5# echo $?
0
```
Запускаем тесты
```
time make -j8 check
```
```
==================
All 9 tests passed
==================
make[2]: Leaving directory '/sources/xz-5.2.5/tests'
make[1]: Leaving directory '/sources/xz-5.2.5/tests'
make[1]: Entering directory '/sources/xz-5.2.5'
make[1]: Leaving directory '/sources/xz-5.2.5'

real    0m3.602s
user    0m4.240s
sys     0m0.632s
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/xz-5.2.5# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf xz-5.2.5
```
---

### 🔷 Zstd-1.5.2
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf zstd-1.5.2.tar.gz && cd zstd-1.5.2
```
Компилируем пакет
```
time make -j8
```
```
real    0m9.723s
user    0m58.707s
sys     0m1.662s
(lfs chroot) root:/sources/zstd-1.5.2# echo $?
0
```
Запускаем тесты
```
time make -j8 check
```
Устанавливаем пакет
```
make prefix=/usr install
```
```
(lfs chroot) root:/sources/zstd-1.5.2# echo $?
0
```
Удаляем статическую библиотеку
```
rm -v /usr/lib/libzstd.a
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf zstd-1.5.2
```
---

### 🔷 File-5.41
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf file-5.41.tar.gz && cd file-5.41
```
Готовим File для компиляции
```
./configure --prefix=/usr
```
Компилируем пакет
```
time make -j8
```
```
real    0m1.050s
user    0m4.315s
sys     0m0.495s
(lfs chroot) root:/sources/file-5.41# echo $?
0
```
Запускаем тесты
```
time make -j8 check
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/file-5.41# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf file-5.41
```
---

### 🔷 Readline-8.1.2
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf readline-8.1.2.tar.gz && cd readline-8.1.2
```
Для того, чтобы при переустановке readline не переименовывались страые библиотеки, запускаем
```
sed -i '/MV.*old/d' Makefile.in
```
```
sed -i '/{OLDSUFF}/c:' support/shlib-install
```
Готовим Readline для компиляции
```
./configure --prefix=/usr --disable-static --with-curses --docdir=/usr/share/doc/readline-8.1.2
```
Компилируем пакет
```
time make -j8 SHLIB_LIBS="-lncursesw"
```
```
real    0m0.755s
user    0m4.366s
sys     0m0.352s
(lfs chroot) root:/sources/readline-8.1.2# echo $?
0
```
Устанавливаем пакет
```
make SHLIB_LIBS="-lncursesw" install
```
```
(lfs chroot) root:/sources/readline-8.1.2# echo $?
0
```
Устанавливаем документацию
```
install -v -m644 doc/*.{ps,pdf,html,dvi} /usr/share/doc/readline-8.1.2
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf readline-8.1.2
```
---

### 🔷 M4-1.4.19
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf m4-1.4.19.tar.xz && cd m4-1.4.19
```
Готовим M4 для компиляции
```
./configure --prefix=/usr
```
Компилируем пакет
```
time make -j8
```
```
real    0m2.865s
user    0m8.584s
sys     0m1.128s
```
Запускаем тесты
```
time make -j8 check
```
```
============================================================================
Testsuite summary for GNU M4 1.4.19
============================================================================
# TOTAL: 267
# PASS:  245
# SKIP:  22
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
make[6]: Leaving directory '/sources/m4-1.4.19/tests'
make[5]: Leaving directory '/sources/m4-1.4.19/tests'
make[4]: Leaving directory '/sources/m4-1.4.19/tests'
make[3]: Leaving directory '/sources/m4-1.4.19/tests'
make[2]: Leaving directory '/sources/m4-1.4.19/tests'
make[1]: Leaving directory '/sources/m4-1.4.19'

real    0m11.322s
user    0m26.028s
sys     0m3.849s
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/m4-1.4.19# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf m4-1.4.19
```
---
### 🔷 Bc-5.2.2
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf bc-5.2.2.tar.xz && cd bc-5.2.2
```
Готовим Bc для компиляции
```
CC=gcc ./configure --prefix=/usr -G -O3
```
Компилируем пакет
```
time make -j8
```
```
real    0m0.818s
user    0m2.487s
sys     0m0.210s
(lfs chroot) root:/sources/bc-5.2.2# echo $?
0
```
Запускаем тесты
```
time make -j8 check
```
```
***********************************************************************
pass
Running bc posix_errors...pass

All bc tests passed.

***********************************************************************

real    0m0.672s
user    0m1.852s
sys     0m0.423s
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/m4-1.4.19# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf bc-5.2.2
```
---

### 🔷 Flex-2.6.4
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf flex-2.6.4.tar.gz && cd flex-2.6.4
```
Готовим Flex для компиляции
```
./configure --prefix=/usr --docdir=/usr/share/doc/flex-2.6.4 --disable-static
```
Компилируем пакет
```
time make -j8
```
```
real    0m1.799s
user    0m6.600s
sys     0m0.617s
(lfs chroot) root:/sources/flex-2.6.4# echo $?
0
```
Запускаем тесты
```
time make -j8 check
```
```
============================================================================
Testsuite summary for the fast lexical analyser generator 2.6.4
============================================================================
# TOTAL: 114
# PASS:  114
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
real    0m4.282s
user    0m27.662s
sys     0m2.578s
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/flex-2.6.4# echo $?
0
```
Создаем симлинк
```
ln -sv flex /usr/bin/lex
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf flex-2.6.4
```
---

### 🔷 Tcl-8.6.12
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf tcl8.6.12-src.tar.gz && cd tcl8.6.12
```
Распаковываем документацию
```
tar -xf ../tcl8.6.12-html.tar.gz --strip-components=1
```
Готовим Tcl для компиляции
```
SRCDIR=$(pwd) && cd unix
```
```
./configure --prefix=/usr --mandir=/usr/share/man $([ "$(uname -m)" = x86_64 ] && echo --enable-64bit)
```
Компилируем пакет
```
time make -j8
```
```
real    0m39.463s
user    1m8.002s
sys     0m4.414s
(lfs chroot) root:/sources/tcl8.6.12/unix# echo $?
0
```
```
sed -e "s|$SRCDIR/unix|/usr/lib|" -e "s|$SRCDIR|/usr/include|" -i tclConfig.sh
```
```
sed -e "s|$SRCDIR/unix/pkgs/tdbc1.1.3|/usr/lib/tdbc1.1.3|" -e "s|$SRCDIR/pkgs/tdbc1.1.3/generic|/usr/include|" -e "s|$SRCDIR/pkgs/tdbc1.1.3/library|/usr/lib/tcl8.6|" -e "s|$SRCDIR/pkgs/tdbc1.1.3|/usr/include|" -i pkgs/tdbc1.1.3/tdbcConfig.sh
```
```
sed -e "s|$SRCDIR/unix/pkgs/itcl4.2.2|/usr/lib/itcl4.2.2|" -e "s|$SRCDIR/pkgs/itcl4.2.2/generic|/usr/include|" -e "s|$SRCDIR/pkgs/itcl4.2.2|/usr/include|" -i pkgs/itcl4.2.2/itclConfig.sh
```
```
unset SRCDIR
```
Запускаем тесты
```
time make -j8 test
```
```
real    3m53.757s
user    0m28.397s
sys     0m6.988s
(lfs chroot) root:/sources/tcl8.6.12/unix# echo $?     
0
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/tcl8.6.12/unix# echo $?
0
```
```
chmod -v u+w /usr/lib/libtcl8.6.so
```
Устанавливаем Tcl заголовки
```
make install-private-headers
```
Создаем необходимый симлинк
```
ln -sfv tclsh8.6 /usr/bin/tclsh
```
Переименовываем страницу руководства, которая конфликтует со страницей руководства Perl
```
mv /usr/share/man/man3/{Thread,Tcl_Thread}.3
```
Устанавливаем загруженную документацию
```
mkdir -v -p /usr/share/doc/tcl-8.6.12
```
```
cp -v -r  ../html/* /usr/share/doc/tcl-8.6.12
```
Удаляем исходные файлы пакета из source
```
cd .. && cd .. && rm -rf tcl8.6.12
```
---

### 🔷 Expect-5.45.4
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf expect5.45.4.tar.gz && cd expect5.45.4
```
Готовим Expect для компиляции
```
./configure --prefix=/usr --with-tcl=/usr/lib --enable-shared --mandir=/usr/share/man --with-tclinclude=/usr/include
```
Компилируем пакет
```
time make -j8
```
```
real    0m0.500s
user    0m2.484s
sys     0m0.240s
(lfs chroot) root:/sources/expect5.45.4# echo $?
0
```
Запускаем тесты
```
time make -j8 test
```
```
real    0m13.062s
user    0m0.045s
sys     0m0.007s
(lfs chroot) root:/sources/expect5.45.4# echo $?
0
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/expect5.45.4# echo $?
0
```
```
ln -svf expect5.45.4/libexpect5.45.4.so /usr/lib
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf expect5.45.4
```
---

### 🔷 DejaGNU-1.6.3
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf dejagnu-1.6.3.tar.gz && cd dejagnu-1.6.3
```
Создаем папку build и переходим в нее
```
mkdir -v build && cd build
```
Готовим DejaGNU для компиляции
```
../configure --prefix=/usr
```
```
makeinfo --html --no-split -o doc/dejagnu.html ../doc/dejagnu.texi
```
```
makeinfo --plaintext -o doc/dejagnu.txt  ../doc/dejagnu.texi
```
Компилируем пакет
```
time make -j8
```
```
real    0m0.004s
user    0m0.004s
sys     0m0.000s
(lfs chroot) root:/sources/dejagnu-1.6.3/build# echo $?
0
```
Запускаем тесты
```
time make -j8 test
```
```
real    0m13.062s
user    0m0.045s
sys     0m0.007s
(lfs chroot) root:/sources/expect5.45.4# echo $?
0
```
Устанавливаем пакет
```
install -v -dm755 /usr/share/doc/dejagnu-1.6.3
```
```
install -v -m644 doc/dejagnu.{html,txt} /usr/share/doc/dejagnu-1.6.3
```
```
(lfs chroot) root:/sources/expect5.45.4# echo $?
0
```
```
ln -svf expect5.45.4/libexpect5.45.4.so /usr/lib
```
Запускаем тесты
```
time make -j8 check
```
Удаляем исходные файлы пакета из source
```
cd .. && cd .. && rm -rf dejagnu-1.6.3
```
---

### 🔷 Binutils-2.38
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf binutils-2.38.tar.xz && cd binutils-2.38
```
Проверяем, что PTY работают внутри окружения chroot
```
expect -c "spawn ls"
```
Устанавливаем патч
```
patch -Np1 -i ../binutils-2.38-lto_fix-1.patch
```
```
sed -e '/R_386_TLS_LE /i \   || (TYPE) == R_386_TLS_IE \\' -i ./bfd/elfxx-x86.h
```
Создаем папку build и переходим в нее
```
mkdir -v build && cd build
```
Готовим Binutils для компиляции
```
../configure --prefix=/usr --enable-gold --enable-ld=default --enable-plugins --enable-shared --disable-werror --enable-64-bit-bfd --with-system-zlib
```
Компилируем пакет
```
time make -j8 tooldir=/usr
```
```
real    1m8.624s
user    6m16.369s
sys     0m27.457s
(lfs chroot) root:/sources/binutils-2.38/build# echo $?
0
```
Запускаем тесты
```
time make -j8 -k check
```
```
============================================================================
Testsuite summary for gold 0.1
============================================================================
# TOTAL: 4
# PASS:  4
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
```
real    0m13.728s
user    1m1.003s
sys     0m9.795s
```
Устанавливаем пакет
```
make tooldir=/usr install
```
```
(lfs chroot) root:/sources/binutils-2.38/build# echo $?
0
```
Удаляем ненужные статические библиотеки
```
rm -fv /usr/lib/lib{bfd,ctf,ctf-nobfd,opcodes}.a
```
Удаляем исходные файлы пакета из source
```
cd .. && cd .. && rm -rf binutils-2.38
```
---

### 🔷 GMP-6.2.1
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf gmp-6.2.1.tar.xz && cd gmp-6.2.1
```
Готовим GMP для компиляции
```
./configure --prefix=/usr --enable-cxx --disable-static --docdir=/usr/share/doc/gmp-6.2.1
```
Компилируем пакет
```
time make -j8
```
```
real    0m6.076s
user    0m27.680s
sys     0m4.053s
(lfs chroot) root:/sources/gmp-6.2.1# echo $?
0
```
Компилируем документацию
```
make html
```
Тестируем результаты
```
time make -j8 check 2>&1 | tee gmp-check-log
```
```
real    0m11.209s
user    0m56.364s
sys     0m5.487s
```
Убеждаемся, что все 197 тестов пройдены
```
awk '/# PASS:/{total+=$3} ; END{print total}' gmp-check-log
```
```
197
```
Устанавливаем пакет и его документацию
```
make install
```
```
(lfs chroot) root:/sources/gmp-6.2.1# echo $?
0
```
```
make install-html
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf gmp-6.2.1
```
---

### 🔷 MPFR-4.1.0
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf mpfr-4.1.0.tar.xz && cd mpfr-4.1.0
```
Готовим MPFR для компиляции
```
./configure --prefix=/usr --disable-static --enable-thread-safe --docdir=/usr/share/doc/mpfr-4.1.0
```
Компилируем пакет
```
time make -j8
```
```
real    0m3.457s
user    0m19.655s
sys     0m2.606s
(lfs chroot) root:/sources/mpfr-4.1.0# echo $?
0
```
Генерируем документацию
```
make html
```
Тестируем результаты
```
time make -j8 check
```
```
============================================================================
Testsuite summary for MPFR 4.1.0
============================================================================
# TOTAL: 183
# PASS:  181
# SKIP:  2
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================

real    0m7.890s
user    0m47.025s
sys     0m4.250s
```
Устанавливаем пакет и его документацию
```
make install
```
```
(lfs chroot) root:/sources/mpfr-4.1.0# echo $?
0
```
```
make install-html
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf mpfr-4.1.0
```
---

### 🔷 MPC-1.2.1
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf mpc-1.2.1.tar.gz && cd mpc-1.2.1
```
Готовим MPC для компиляции
```
./configure --prefix=/usr --disable-static --docdir=/usr/share/doc/mpc-1.2.1
```
Компилируем пакет
```
time make -j8
```
```
real    0m0.886s
user    0m4.201s
sys     0m0.682s
(lfs chroot) root:/sources/mpc-1.2.1# echo $?
0
```
Генерируем документацию
```
make html
```
Тестируем результаты
```
time make -j8 check
```
```
============================================================================
Testsuite summary for mpc 1.2.1
============================================================================
# TOTAL: 69
# PASS:  69
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================

real    0m3.595s
user    0m19.146s
sys     0m1.616s
```
Устанавливаем пакет и его документацию
```
make install
```
```
(lfs chroot) root:/sources/mpc-1.2.1# echo $?
0
```
```
make install-html
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf mpc-1.2.1
```
---

### 🔷 Attr-2.5.1
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf attr-2.5.1.tar.gz && cd attr-2.5.1 
```
Готовим Attr для компиляции
```
./configure --prefix=/usr --disable-static --sysconfdir=/etc --docdir=/usr/share/doc/attr-2.5.1
```
Компилируем пакет
```
time make -j8
```
```
real    0m0.354s
user    0m1.489s
sys     0m0.269s
(lfs chroot) root:/sources/attr-2.5.1# echo $?
0
```
Тестируем результаты
```
time make -j8 check
```
```
============================================================================
Testsuite summary for attr 2.5.1
============================================================================
# TOTAL: 2
# PASS:  2
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================

real    0m0.569s
user    0m0.097s
sys     0m0.031s
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/attr-2.5.1# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf attr-2.5.1
```
---

### 🔷 Acl-2.3.1
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf acl-2.3.1.tar.xz && cd acl-2.3.1
```
Готовим Acl для компиляции
```
./configure --prefix=/usr --disable-static --docdir=/usr/share/doc/acl-2.3.1
```
Компилируем пакет
```
time make -j8
```
```
real    0m0.918s
user    0m3.972s
sys     0m0.736s
(lfs chroot) root:/sources/acl-2.3.1# echo $?
0
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/acl-2.3.1# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf acl-2.3.1
```
---

### 🔷 Libcap-2.63
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf libcap-2.63.tar.xz && cd libcap-2.63
```
Упреждаем установку статических библиотек
```
sed -i '/install -m.*STA/d' libcap/Makefile
```
Компилируем пакет
```
time make -j8 prefix=/usr lib=lib
```
```
real    0m0.723s
user    0m1.081s
sys     0m0.161s
(lfs chroot) root:/sources/libcap-2.63# echo $?
0
```
Тестируем пакет
```
time make -j8 test
```
```
real    0m0.175s
user    0m0.180s
sys     0m0.041s
(lfs chroot) root:/sources/libcap-2.63# echo $?
0
```
Устанавливаем пакет
```
make prefix=/usr lib=lib install
```
```
(lfs chroot) root:/sources/libcap-2.63# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf libcap-2.63
```
---

### 🔷 Shadow-4.11.1
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf shadow-4.11.1.tar.xz && cd shadow-4.11.1
```
Отключаем установку программы groups и ее man страниц, так как Coreutils обеспечивает версию лучше. Также отключаем установку man страниц, которые уже были установлены ранее.
```
sed -i 's/groups$(EXEEXT) //' src/Makefile.in
find man -name Makefile.in -exec sed -i 's/groups\.1 / /'   {} \;
find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \;
find man -name Makefile.in -exec sed -i 's/passwd\.5 / /'   {} \;
```
Instead of using the default crypt method, use the more secure SHA-512 method of password encryption, which also allows passwords longer than 8 characters. It is also necessary to change the obsolete /var/spool/mail location for user mailboxes that Shadow uses by default to the /var/mail location used currently. And, get rid of /bin and /sbin from PATH, since they are simply symlinks to their counterpart in /usr.
Вместо использования криптографического метода, предоставляемого по умолчанию, ворспользуемся болле эффектиынм SHA-512 для шифрования паролей, который также разрешает пароли длиннее 8 символов. 
```
sed -e 's:#ENCRYPT_METHOD DES:ENCRYPT_METHOD SHA512:' -e 's:/var/spool/mail:/var/mail:' -e '/PATH=/{s@/sbin:@@;s@/bin:@@}' -i etc/login.defs
```
Готовим Shadow для компиляции
```
touch /usr/bin/passwd
```
```
./configure --sysconfdir=/etc --disable-static --with-group-name-max-length=32
```
Компилируем пакет
```
time make -j8
```
```
real    0m2.197s
user    0m11.442s
sys     0m1.859s
(lfs chroot) root:/sources/shadow-4.11.1# echo $?
0
```
Устанавливаем пакет
```
make exec_prefix=/usr install
```
```
(lfs chroot) root:/sources/shadow-4.11.1# echo $?
0
```
```
make -C man install-man
```
```
(lfs chroot) root:/sources/shadow-4.11.1# echo $?
0
```
Конфигурируем Shadow.
Для включения теневых паролей запускаем
```
pwconv
```
Для включения теневых групп паролей запускаем
```
grpconv
```
Меняем параметры по умолчанию
```
mkdir -p /etc/default
useradd -D --gid 999
```
Устанавливаем пароль root
```
passwd root
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf shadow-4.11.1
```
---

### 🔷 GCC-11.2.0
Распаковываем исходники и переходим в папку с пакетом
```
tar xvf gcc-11.2.0.tar.xz && cd gcc-11.2.0
```
Отключаем установку программы groups и ее man страниц, так как Coreutils обеспечивает версию лучше. Также отключаем установку man страниц, которые уже были установлены ранее.
```
sed -i 's/groups$(EXEEXT) //' src/Makefile.in
find man -name Makefile.in -exec sed -i 's/groups\.1 / /'   {} \;
find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \;
find man -name Makefile.in -exec sed -i 's/passwd\.5 / /'   {} \;
```
Instead of using the default crypt method, use the more secure SHA-512 method of password encryption, which also allows passwords longer than 8 characters. It is also necessary to change the obsolete /var/spool/mail location for user mailboxes that Shadow uses by default to the /var/mail location used currently. And, get rid of /bin and /sbin from PATH, since they are simply symlinks to their counterpart in /usr.
Вместо использования криптографического метода, предоставляемого по умолчанию, ворспользуемся болле эффектиынм SHA-512 для шифрования паролей, который также разрешает пароли длиннее 8 символов. 
```
sed -e 's:#ENCRYPT_METHOD DES:ENCRYPT_METHOD SHA512:' -e 's:/var/spool/mail:/var/mail:' -e '/PATH=/{s@/sbin:@@;s@/bin:@@}' -i etc/login.defs
```
Готовим Shadow для компиляции
```
touch /usr/bin/passwd
```
```
./configure --sysconfdir=/etc --disable-static --with-group-name-max-length=32
```
Компилируем пакет
```
time make -j8
```
```
real    0m2.197s
user    0m11.442s
sys     0m1.859s
(lfs chroot) root:/sources/shadow-4.11.1# echo $?
0
```
Устанавливаем пакет
```
make exec_prefix=/usr install
```
```
(lfs chroot) root:/sources/shadow-4.11.1# echo $?
0
```
```
make -C man install-man
```
```
(lfs chroot) root:/sources/shadow-4.11.1# echo $?
0
```
Конфигурируем Shadow.
Для включения теневых паролей запускаем
```
pwconv
```
Для включения теневых групп паролей запускаем
```
grpconv
```
Меняем параметры по умолчанию
```
mkdir -p /etc/default
useradd -D --gid 999
```
Устанавливаем пароль root
```
passwd root
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf shadow-4.11.1
```
---

















