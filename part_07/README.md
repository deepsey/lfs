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
Собираем и устанавливаем пакет
```
make install
```
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
Исправляем проблему поломки libasan.a, сбилдив этот пакет при помощи Glibc-2.34 или более поздней:
```
sed -e '/static.*SIGSTKSZ/d' -e 's/return kAltStackSize/return SIGSTKSZ * 4/' -i libsanitizer/sanitizer_common/sanitizer_posix_libcdep.cpp
```
Если собираем в системе x86_64, то меняем наименовние директории для 64-х битных библиотек на “lib”
```
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' -i.orig gcc/config/i386/t-linux64
  ;;
esac
```
The GCC documentation recommends building GCC in a dedicated build directory:
Сборку GCC рекомендуется проводить в специальной директории build
```
mkdir -v build && cd build
```
Готовим GCC для компиляции
```
../configure --prefix=/usr LD=ld --enable-languages=c,c++ --disable-multilib --disable-bootstrap --with-system-zlib
```
Компилируем пакет
```
time make -j8
```
```
real    4m15.907s
user    20m2.298s
sys     1m28.510s
(lfs chroot) root:/sources/gcc-11.2.0/build# echo $?
0
```
Запускаем тесты.
Увеличиваем размер стека для запуска тестов
```
ulimit -s 32768
```
Тесты запускаем от непривилегированного пользователя
```
chown -Rv tester .
```
```
su tester -c "PATH=$PATH make -k -j8 check"
```
Для получения суммарной сводки результатов тестов запустим
```
../contrib/test_summary
```
Для вывода только сводки по тестам можно запустить
```
../contrib/test_summary | grep -A7 Summ
```

Устанавливаем пакет и удаляем ненужные директории
```
make install
```
```
(lfs chroot) root:/sources/gcc-11.2.0/build# echo $?
0
```
```
rm -rf /usr/lib/gcc/$(gcc -dumpmachine)/11.2.0/include-fixed/bits/
```
Меняем владельца директорий на root:root для корректной работы
```
chown -v -R root:root /usr/lib/gcc/*linux-gnu/11.2.0/include{,-fixed}
```
Создаем симлинк, необходимый по исторически сложившейся структкре FHS
```
ln -svr /usr/bin/cpp /usr/lib
```
Добавляем совместимый симлинк для включения сборочных программ при помощи Link Time Optimization (LTO)
```
ln -sfv ../../libexec/gcc/$(gcc -dumpmachine)/11.2.0/liblto_plugin.so /usr/lib/bfd-plugins/
```
Теперь, когда финальный набор инструментов находится на своем месте, важно еще раз убедиться, чт о компиляция и линковка работают так, как одидается. Запускаем для этого несколько тестов на отсутствие тривиальных ошибок
```
echo 'int main(){}' > dummy.c
cc dummy.c -v -Wl,--verbose &> dummy.log
readelf -l a.out | grep ': /lib'
```
```
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
(lfs chroot) root:/sources/gcc-11.2.0/build# echo $?
0
```
Ошибок быть не должно, и вывод последней команды должен пбыть примерно таким (в зависимости от платформы может отличаться имя линкера)
```
[Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```
Теперь убедимся, что мы установили для использования корректные стартовые файлы
```
grep -o '/usr/lib.*/crt[1in].*succeeded' dummy.log
```
```
/usr/lib/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../lib/crt1.o succeeded
/usr/lib/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../lib/crti.o succeeded
/usr/lib/gcc/x86_64-pc-linux-gnu/11.2.0/../../../../lib/crtn.o succeeded
(lfs chroot) root:/sources/gcc-11.2.0/build# echo $?
0
```
Вывод последней команды должен быть как указано выше.  
Проверяем, что компилятор ищет корректные заголовочные файлы
```
grep -B4 '^ /usr/include' dummy.log
```
```
 /usr/lib/gcc/x86_64-pc-linux-gnu/11.2.0/include
 /usr/local/include
 /usr/lib/gcc/x86_64-pc-linux-gnu/11.2.0/include-fixed
 /usr/include
(lfs chroot) root:/sources/gcc-11.2.0/build# echo $?
0
```
Вывод команды должен быть как указано выше
Далее проверяем, что новый линкер используется с корректными путями поиска
```
grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'
```
```
SEARCH_DIR("/usr/x86_64-pc-linux-gnu/lib64")
SEARCH_DIR("/usr/local/lib64")
SEARCH_DIR("/lib64")
SEARCH_DIR("/usr/lib64")
SEARCH_DIR("/usr/x86_64-pc-linux-gnu/lib")
SEARCH_DIR("/usr/local/lib")
SEARCH_DIR("/lib")
SEARCH_DIR("/usr/lib");
(lfs chroot) root:/sources/gcc-11.2.0/build# echo $?
0
```
Вывод должен быть примерно как выше.  
Далее проверяем, правильную ли libc мы используем
```
grep "/lib.*/libc.so.6 " dummy.log
```
```
(lfs chroot) root:/sources/gcc-11.2.0/build# echo $?
0
```
Убеждаемся, что GCC использует правильный динамический линкер
```
grep found dummy.log
```
```
found ld-linux-x86-64.so.2 at /usr/lib/ld-linux-x86-64.so.2
(lfs chroot) root:/sources/gcc-11.2.0/build# echo $?
0
```
Для платформы x86-64 вывод должен быть как указано выше.  
Убедившись, что все работает корректно, очищаем тестовые файлы
```
rm -v dummy.c a.out dummy.log
```
Перемещаем несоответствующие файлы
```
mkdir -pv /usr/share/gdb/auto-load/usr/lib
```
```
mv -v /usr/lib/*gdb.py /usr/share/gdb/auto-load/usr/lib
```
Удаляем исходные файлы пакета из source
```
cd .. && cd .. && rm -rf gcc-11.2.0
```
---

### 🔷 8.27. Pkg-config-0.29.2

Распаковываем Pkg-config-0.29.2 и переходим в папку с пакетом
```
tar xvf pkg-config-0.29.2.tar.gz && cd pkg-config-0.29.2
```
Готовим Pkg-config для компиляции
```
./configure --prefix=/usr --with-internal-glib --disable-host-tool --docdir=/usr/share/doc/pkg-config-0.29.2
```
Компилируем пакет
```
time make -j8
```
```
real    0m3.221s
user    0m16.844s
sys     0m1.603s
(lfs chroot) root:/sources/pkg-config-0.29.2# echo $?
0
```
Тестируем результаты
```
make check
```
```
===================
All 30 tests passed
===================
make[2]: Leaving directory '/sources/pkg-config-0.29.2/check'
make[1]: Leaving directory '/sources/pkg-config-0.29.2/check'
(lfs chroot) root:/sources/pkg-config-0.29.2# echo $?
0
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/pkg-config-0.29.2# echo $?
0
```

Удаляем исходные файлы пакета из source

```
cd .. && rm -rf pkg-config-0.29.2
```
---

### 🔷 8.28. Ncurses-6.3

Распаковываем Ncurses-6.3 и переходим в папку с пакетом
```
tar xvf ncurses-6.3.tar.gz && cd ncurses-6.3
```
Готовим Ncurses для компиляции
```
./configure --prefix=/usr --mandir=/usr/share/man --with-shared --without-debug --without-normal --enable-pc-files --enable-widec --with-pkg-config-libdir=/usr/lib/pkgconfig
```
Компилируем пакет
```
time make -j8
```
```
real    0m5.597s
user    0m26.036s
sys     0m3.513s
(lfs chroot) root:/sources/ncurses-6.3# echo $?
0
```
Инсталляция этого пакета перезапишет libncursesw.so.6.3 в месте расположения. Это может привести к нарушению в работе процесса shellб который использует код и данные этой библиотеки. Установим пакет с переменной DESTDIR и корректно переместим файл библиотеки, используя команду install. Не нужный статический архив также удаляем
```
make DESTDIR=$PWD/dest install
```
```
install -vm755 dest/usr/lib/libncursesw.so.6.3 /usr/lib
```
```
rm -v  dest/usr/lib/{libncursesw.so.6.3,libncurses++w.a}
```
```
cp -av dest/* /
```
Создаем нужные симлинки
```
for lib in ncurses form panel menu ; do
    rm -vf                    /usr/lib/lib${lib}.so
    echo "INPUT(-l${lib}w)" > /usr/lib/lib${lib}.so
    ln -sfv ${lib}w.pc        /usr/lib/pkgconfig/${lib}.pc
done
```
Убеждаемся, что старые приложения, который ищут -lcurses во время компиляции, все еще компилируемые:
```
rm -vf /usr/lib/libcursesw.so
```
```
echo "INPUT(-lncursesw)" > /usr/lib/libcursesw.so
```
```
ln -sfv libncurses.so /usr/lib/libcurses.so
```
Устанавливаем документацию
```
mkdir -pv /usr/share/doc/ncurses-6.3
```
```
cp -v -R doc/* /usr/share/doc/ncurses-6.3
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf ncurses-6.3
```

---

### 🔷 8.29. Sed-4.8

Распаковываем Sed-4.8 и переходим в папку с пакетом
```
tar xvf sed-4.8.tar.xz && cd sed-4.8
```
Конфигурируем Ncurses для компиляции
```
./configure --prefix=/usr
```
Компилируем пакет и генерируем HTML документацию
```
time make -j8
```
```
real    0m1.985s
user    0m4.977s
sys     0m0.555s
(lfs chroot) root:/sources/sed-4.8# echo $?
0
```
```
make html
```
Выпрлняем следующие действия для тестирования результатов
```
chown -Rv tester .
```
```
su tester -c "PATH=$PATH make check"
```
```
============================================================================
Testsuite summary for GNU sed 4.8
============================================================================
# TOTAL: 178
# PASS:  157
# SKIP:  21
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
===========================================================================
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/sed-4.8# echo $?
0
```
```
install -d -m755 /usr/share/doc/sed-4.8
```
```
install -m644 doc/sed.html /usr/share/doc/sed-4.8
```
Удаляем исходные файлы пакета из source

```
cd .. && rm -rf sed-4.8
```

---

### 🔷 8.30. Psmisc-23.4
Распаковываем Psmisc-23.4 и переходим в папку с пакетом
```
tar xvf psmisc-23.4.tar.xz && cd psmisc-23.4
```
Конфигурируем Psmisc для компиляции
```
./configure --prefix=/usr
```
Компилируем пакет
```
time make -j8
```
```
real    0m0.372s
user    0m1.032s
sys     0m0.126s
(lfs chroot) root:/sources/psmisc-23.4# echo $?
0
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/psmisc-23.4# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf psmisc-23.4
```

---

### 🔷 8.31. Gettext-0.21
Распаковываем Gettext-0.21 и переходим в папку с пакетом
```
tar xvf gettext-0.21.tar.xz && cd gettext-0.21 
```
Конфигурируем Gettext для компиляции
```
./configure --prefix=/usr --disable-static --docdir=/usr/share/doc/gettext-0.21
```
Компилируем пакет
```
time make -j8
```
```
real    1m0.511s
user    1m50.590s
sys     0m12.377s
(lfs chroot) root:/sources/gettext-0.21# echo $?
0
```
Тестируем результаты
```
make -j8 check
```
```
============================================================================
Testsuite summary for gettext-tools 0.21
============================================================================
# TOTAL: 266
# PASS:  252
# SKIP:  14
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/gettext-0.21# echo $?
0
```
```
chmod -v 0755 /usr/lib/preloadable_libintl.so
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf gettext-0.21
```

---
### 🔷 8.32. Bison-3.8.2
Распаковываем Bison-3.8.2 и переходим в папку с пакетом
```
tar xvf bison-3.8.2.tar.xz && cd bison-3.8.2
```
Конфигурируем Bison для компиляции
```
./configure --prefix=/usr --docdir=/usr/share/doc/bison-3.8.2
```
Компилируем пакет
```
time make -j8
```
```
real    0m1.982s
user    0m12.243s
sys     0m1.357s
(lfs chroot) root:/sources/bison-3.8.2# echo $?
0
```
Тестируем результаты
```
make -j8 check
```
```
## ------------- ##
## Test results. ##
## ------------- ##

712 tests were successful.
64 tests were skipped.
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/bison-3.8.2# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf bison-3.8.2
```

---

### 🔷 8.33. Grep-3.7
Распаковываем Grep-3.7 и переходим в папку с пакетом
```
tar xvf grep-3.7.tar.xz && cd grep-3.7
```
Конфигурируем Grep для компиляции
```
./configure --prefix=/usr
```
Компилируем пакет
```
time make -j8
```
```
real    0m1.562s
user    0m5.647s
sys     0m0.778s
(lfs chroot) root:/sources/grep-3.7# echo $?
0
```
Тестируем результаты
```
make -j8 check
```
```
============================================================================
Testsuite summary for GNU grep 3.7
============================================================================
# TOTAL: 201
# PASS:  192
# SKIP:  9
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/grep-3.7# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf grep-3.7
```

---

### 🔷 8.34. Bash-5.1.16
Распаковываем Bash-5.1.16 и переходим в папку с пакетом
```
tar xvf bash-5.1.16.tar.gz && cd bash-5.1.16
```
Конфигурируем Bash для компиляции
```
./configure --prefix=/usr --docdir=/usr/share/doc/bash-5.1.16 --without-bash-malloc --with-installed-readline
```
Компилируем пакет
```
time make -j8
```
```
real    0m3.329s
user    0m17.781s
sys     0m1.613s
(lfs chroot) root:/sources/bash-5.1.16# echo $?
0
```
Подготавливаем ользователя для проведения тестов
```
chown -Rv tester .
```
Тестируем результаты
```
su -s /usr/bin/expect tester << EOF
set timeout -1
spawn make tests
expect eof
lassign [wait] _ _ _ value
exit $value
EOF
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/bash-5.1.16# echo $?
0
```
Запускаем заново скомпилированный bash (вместо текущего уже запущенного)
```
exec /usr/bin/bash --login
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf bash-5.1.16
```

---

### 🔷 8.35. Libtool-2.4.6
Распаковываем Libtool-2.4.6 и переходим в папку с пакетом
```
tar xvf libtool-2.4.6.tar.xz && cd libtool-2.4.6
```
Конфигурируем Libtool для компиляции
```
./configure --prefix=/usr
```
Компилируем пакет
```
time make -j8
```
```
real    0m0.764s
user    0m1.390s
sys     0m0.173s
(lfs chroot) root:/sources/libtool-2.4.6# echo $?
0
```
Тестируем результаты
```
make -j8 check
```
```
## ------------- ##
## Test results. ##
## ------------- ##

ERROR: 138 tests were run,
64 failed (59 expected failures).
32 tests were skipped.
## -------------------------- ##
## testsuite.log was created. ##
## -------------------------- ##

Please send `tests/testsuite.log' and all information you think might help:

   To: <bug-libtool@gnu.org>
   Subject: [GNU Libtool 2.4.6] testsuite: 123 124 125 126 130 failed

```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/libtool-2.4.6# echo $?
0
```
Удаляем неспользуемую статическую библиотеку
```
rm -fv /usr/lib/libltdl.a
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf libtool-2.4.6
```

---

### 🔷 8.36. GDBM-1.23
Распаковываем GDBM-1.23 и переходим в папку с пакетом
```
tar xvf gdbm-1.23.tar.gz && cd gdbm-1.23 
```
Конфигурируем GDBM для компиляции
```
./configure --prefix=/usr --disable-static --enable-libgdbm-compat
```
Компилируем пакет
```
time make -j8
```
```
real    0m1.174s
user    0m4.714s
sys     0m0.615s
(lfs chroot) root:/sources/gdbm-1.23# echo $?
0
```
Тестируем результаты
```
make check TESTSUITEFLAGS=-j8
```
```
## ------------- ##
## Test results. ##
## ------------- ##

All 33 tests were successful.
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/gdbm-1.23# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf gdbm-1.23
```
---

### 🔷 8.37. Gperf-3.1
Распаковываем Gperf-3.1 и переходим в папку с пакетом
```
tar xvf gperf-3.1.tar.gz && cd gperf-3.1
```
Конфигурируем Gperf для компиляции
```
./configure --prefix=/usr --docdir=/usr/share/doc/gperf-3.1
```
Компилируем пакет
```
time make -j8
```
```
real    0m0.548s
user    0m1.409s
sys     0m0.133s
(lfs chroot) root:/sources/gperf-3.1# echo $?
0
```
Тестируем результаты
```
make -j1 check
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/gperf-3.1# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf gperf-3.1
```

---

### 🔷 8.38. Expat-2.4.6
Распаковываем Expat-2.4.6 и переходим в папку с пакетом
```
tar xvf expat-2.4.6.tar.gz && cd expat-2.4.6
```
Конфигурируем Expat для компиляции
```
./configure --prefix=/usr --disable-static --docdir=/usr/share/doc/expat-2.4.6
```
Компилируем пакет
```
time make -j8
```
```
real    0m1.927s
user    0m3.391s
sys     0m0.180s
(lfs chroot) root:/sources/expat-2.4.6# echo $?
0
```
Тестируем результаты
```
make check TESTSUITEFLAGS=-j8
```
```
============================================================================
Testsuite summary for expat 2.4.6
============================================================================
# TOTAL: 2
# PASS:  2
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/expat-2.4.6# echo $?
0
```
Устанавливаем документацию
```
install -v -m644 doc/*.{html,css} /usr/share/doc/expat-2.4.6
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf expat-2.4.6
```

---

### 🔷 8.39. Inetutils-2.2
Распаковываем Inetutils-2.2 и переходим в папку с пакетом
```
tar xvf inetutils-2.2.tar.xz && cd inetutils-2.2
```
Конфигурируем Inetutils для компиляции
```
./configure --prefix=/usr --bindir=/usr/bin --localstatedir=/var --disable-logger --disable-whois --disable-rcp --disable-rexec --disable-rlogin --disable-rsh --disable-servers
```
Компилируем пакет
```
time make -j8
```
```
real    0m2.754s
user    0m8.815s
sys     0m1.362s
(lfs chroot) root:/sources/inetutils-2.2# echo $?
0
```
Тестируем результаты
```
make check TESTSUITEFLAGS=-j8
```
```
============================================================================
Testsuite summary for GNU inetutils 2.2
============================================================================
# TOTAL: 11
# PASS:  10
# SKIP:  1
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/inetutils-2.2# echo $?
0
```
Перемещаем программу в правильную директорию
```
mv -v /usr/{,s}bin/ifconfig
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf inetutils-2.2
```

---

### 🔷 8.40. Less-590
Распаковываем Less-590 и переходим в папку с пакетом
```
tar xvf less-590.tar.gz && cd less-590
```
Конфигурируем Less для компиляции
```
./configure --prefix=/usr --sysconfdir=/etc
```
Компилируем пакет
```
time make -j8
```
```
real    0m0.527s
user    0m2.986s
sys     0m0.301s
(lfs chroot) root:/sources/less-590# echo $?
0
```
Устанавливаем пакет
```
make install
```
```
(lfs chroot) root:/sources/less-590# echo $?
0
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf less-590
```

---

### 🔷 8.41. Perl-5.34.0
Распаковываем Perl-5.34.0 и переходим в папку с пакетом
```
tar xvf perl-5.34.0.tar.xz && cd perl-5.34.0
```
Устанавливаем патч, исправляющий проблему, на которую указывают последние версии gdbm
```
patch -Np1 -i ../perl-5.34.0-upstream_fixes-1.patch
```
В этой версии Perl теперь выполняется сборка модулей Compress::Raw::Zlib и Compress::Raw::BZip2. По умолчанию для сборки Perl использует встроенную копию исходного кода. Выполните следующую команду, чтобы Perl использовал библиотеки, установленные в системе:
```
export BUILD_ZLIB=False
export BUILD_BZIP2=0
```
Чтобы полностью контролировать процесс настройки Perl, вы можете удалить опции «-des» из приведенной ниже команды и самостоятельно выбрать параметры сборки пакета. В качестве альтернативы можно использовать команду в точности так, как она указана ниже, чтобы применить настройки по умолчанию, которые Perl определяет автоматически:
```
sh Configure -des -Dprefix=/usr -Dvendorprefix=/usr -Dprivlib=/usr/lib/perl5/5.34/core_perl -Darchlib=/usr/lib/perl5/5.34/core_perl -Dsitelib=/usr/lib/perl5/5.34/site_perl -Dsitearch=/usr/lib/perl5/5.34/site_perl -Dvendorlib=/usr/lib/perl5/5.34/vendor_perl -Dvendorarch=/usr/lib/perl5/5.34/vendor_perl -Dman1dir=/usr/share/man/man1 -Dman3dir=/usr/share/man/man3 -Dpager="/usr/bin/less -isR" -Duseshrplib -Dusethreads
```
Компилируем пакет
```
time make -j8
```
```
real    0m23.312s
user    2m4.730s
sys     0m8.555s
(lfs chroot) root:/sources/perl-5.34.0# echo $?
0
```
Для проверки результатов (примерно 11 SBU) выполняем команду:
```
make check TESTSUITEFLAGS=-j8
```
```
Failed 9 tests out of 2542, 99.65% okay.
```
Устанавливаем пакет и выполняем очистку:
```
make install
```
```
(lfs chroot) root:/sources/perl-5.34.0# echo $?
0
```
```
unset BUILD_ZLIB BUILD_BZIP2
```
Удаляем исходные файлы пакета из source
```
cd .. && rm -rf perl-5.34.0
```
---

42
### 🔷 XML::Parser-2.46
Распаковываем XML::Parser-2.46 и переходим в папку с пакетом
```
tar xvf XML-Parser-2.46.tar.gz && cd XML-Parser-2.46
```
TEXT
```
perl Makefile.PL
```
TEXT
```
time make -j8
```
```
real    0m1.038s
user    0m1.155s
sys     0m0.083s
(lfs chroot) root:/sources/XML-Parser-2.46# echo $?
0
```
TEXT
```
make -j8 test
```
```
All tests successful.
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/XML-Parser-2.46# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf XML-Parser-2.46
```
---

43
### 🔷 Intltool-0.51.0
Распаковываем Intltool-0.51.0 и переходим в папку с пакетом
```
tar xvf intltool-0.51.0.tar.gz && cd intltool-0.51.0 
```
TEXT
```
sed -i 's:\\\${:\\\$\\{:' intltool-update.in
```
TEXT
```
./configure --prefix=/usr
```
TEXT
```
time make -j8
```
```
real    0m0.016s
user    0m0.019s
sys     0m0.004s
(lfs chroot) root:/sources/intltool-0.51.0# echo $?
0
```
TEXT
```
make -j8 check
```
```
===========================================================================
Testsuite summary for intltool 0.51.0
============================================================================
# TOTAL: 1
# PASS:  1
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/intltool-0.51.0# echo $?
0
```
```
install -v -Dm644 doc/I18N-HOWTO /usr/share/doc/intltool-0.51.0/I18N-HOWTO
```
TEXT REMOVE
```
cd .. && rm -rf intltool-0.51.0
```
---

44
### 🔷 Autoconf-2.71
Распаковываем Autoconf-2.71 и переходим в папку с пакетом
```
tar xvf autoconf-2.71.tar.xz && cd autoconf-2.71
```
TEXT
```
./configure --prefix=/usr
```
TEXT
```
time make -j8
```
```
real    0m0.160s
user    0m0.523s
sys     0m0.067s
(lfs chroot) root:/sources/autoconf-2.71# echo $?
0
```
TEXT
```
make -j8 check
```
```
543 tests behaved as expected.
56 tests were skipped.
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/autoconf-2.71# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf autoconf-2.71
```
---

45
### 🔷 Automake-1.16.5
Распаковываем Automake-1.16.5 и переходим в папку с пакетом
```
tar xvf automake-1.16.5.tar.xz && cd automake-1.16.5
```
TEXT
```
./configure --prefix=/usr --docdir=/usr/share/doc/automake-1.16.5
```
TEXT
```
time make -j8
```
```
real    0m0.198s
user    0m0.480s
sys     0m0.068s
(lfs chroot) root:/sources/automake-1.16.5# echo $?
0
```
TEXT
```
make -j8 check
```
```
============================================================================
Testsuite summary for GNU Automake 1.16.5
============================================================================
# TOTAL: 2926
# PASS:  2715
# SKIP:  173
# XFAIL: 38
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/automake-1.16.5# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf automake-1.16.5
```
---

46
### 🔷 OpenSSL-3.0.12
В книге устанавливается OpenSSL-3.0.1, но его тестирование заканчивалось с ошибками, поэтому разворачивался OpenSSL-3.0.12
```
wget https://github.com/openssl/openssl/releases/download/openssl-3.0.12/openssl-3.0.12.tar.gz --directory-prefix=$LFS/sources
```
Распаковываем OpenSSL-3.0.12 и переходим в папку с пакетом
```
tar xvf openssl-3.0.12.tar.gz && cd openssl-3.0.12
```
TEXT
```
./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib shared zlib-dynamic
```
TEXT
```
time make -j8
```
```
real    0m0.368s
user    0m0.346s
sys     0m0.026s
(lfs chroot) root:/sources/openssl-3.0.12# echo $?
0
```
TEXT
```
make -j8 test
```
```
All tests successful.
Files=250, Tests=3351, 216 wallclock secs ( 2.81 usr  0.32 sys + 182.36 cusr 24.50 csys = 209.99 CPU)
Result: PASS
```
TEXT
```
sed -i '/INSTALL_LIBS/s/libcrypto.a libssl.a//' Makefile
```
```
make MANSUFFIX=ssl install
```
```
(lfs chroot) root:/sources/openssl-3.0.12# echo $?
0
```
TEXT
```
mv -v /usr/share/doc/openssl /usr/share/doc/openssl-3.0.12
```
TEXT
```
cp -vfr doc/* /usr/share/doc/openssl-3.0.12
```
TEXT REMOVE
```
cd .. && rm -rf openssl-3.0.12
```
---

47
### 🔷 Kmod-29
Распаковываем Kmod-29 и переходим в папку с пакетом
```
tar xvf kmod-29.tar.xz && cd kmod-29 
```
TEXT
```
./configure --prefix=/usr --sysconfdir=/etc --with-openssl --with-xz --with-zstd --with-zlib
```
TEXT
```
time make -j8
```
```
real    0m0.885s
user    0m3.520s
sys     0m0.413s
(lfs chroot) root:/sources/kmod-29# echo $?
0
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/kmod-29# echo $?
0
```
```
for target in depmod insmod modinfo modprobe rmmod; do
  ln -sfv ../bin/kmod /usr/sbin/$target
done
```
```
ln -sfv kmod /usr/bin/lsmod
```
TEXT REMOVE
```
cd .. && rm -rf kmod-29
```
---

48
### 🔷 Libelf from Elfutils-0.186
Распаковываем Elfutils-0.186 и переходим в папку с пакетом
```
tar xvf elfutils-0.186.tar.bz2 && cd elfutils-0.186
```
TEXT
```
./configure --prefix=/usr --disable-debuginfod --enable-libdebuginfod=dummy
```
TEXT
```
time make -j8
```
```
real    0m11.520s
user    0m54.303s
sys     0m9.094s
(lfs chroot) root:/sources/elfutils-0.186# echo $?
0
```
TEXT
```
make check
```
```
============================================================================
Testsuite summary for elfutils 0.186
============================================================================
# TOTAL: 232
# PASS:  227
# SKIP:  5
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
make -C libelf install
```
```
(lfs chroot) root:/sources/elfutils-0.186# echo $?
0
```
```
install -vm644 config/libelf.pc /usr/lib/pkgconfig
```
```
rm /usr/lib/libelf.a
```
TEXT REMOVE
```
cd .. && rm -rf elfutils-0.186
---

49
### 🔷 Libffi-3.4.2
Распаковываем Libffi-3.4.2 и переходим в папку с пакетом
```
tar xvf libffi-3.4.2.tar.gz && cd libffi-3.4.2
```
TEXT
```
./configure --prefix=/usr --disable-static --with-gcc-arch=native --disable-exec-static-tramp
```
TEXT
```
time make -j8
```
```
real    0m0.453s
user    0m0.989s
sys     0m0.130s
(lfs chroot) root:/sources/libffi-3.4.2# echo $?
0
```
TEXT
```
make check
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/libffi-3.4.2# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf libffi-3.4.2

---

50
### 🔷 Python-3.10.2
Распаковываем Python-3.10.2 и переходим в папку с пакетом
```
tar xvf Python-3.10.2.tar.xz && cd Python-3.10.2
```
TEXT
```
./configure --prefix=/usr --enable-shared --with-system-expat --with-system-ffi -with-ensurepip=yes --enable-optimizations
```
TEXT
```
time make -j8
```
```
real    2m18.905s
user    5m36.379s
sys     0m13.204s
(lfs chroot) root:/sources/Python-3.10.2# echo $?
0
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/Python-3.10.2# echo $?
0
```
TEXT
```
install -v -dm755 /usr/share/doc/python-3.10.2/html
```
```
tar --strip-components=1 --no-same-owner --no-same-permissions -C /usr/share/doc/python-3.10.2/html -xvf ../python-3.10.2-docs-html.tar.bz2
```
TEXT REMOVE
```
cd .. && rm -rf Python-3.10.2

---

51
### 🔷 Ninja-1.10.2
Распаковываем Ninja-1.10.2 и переходим в папку с пакетом
```
tar xvf ninja-1.10.2.tar.gz && cd ninja-1.10.2
```
TEXT
```
export NINJAJOBS=8
```
TEXT
```
sed -i '/int Guess/a int   j = 0; char* jobs = getenv( "NINJAJOBS" ); if ( jobs != NULL ) j = atoi( jobs ); if ( j > 0 ) return j; ' src/ninja.cc
```
TEXT
```
python3 configure.py --bootstrap
```
TEXT
```
./ninja ninja_test
```
```
[19/19] LINK ninja_test
(lfs chroot) root:/sources/ninja-1.10.2# echo $?
0
```
```
./ninja_test --gtest_filter=-SubprocessTest.SetWithLots
```
```
[343/343] ElideMiddle.ElideInTheMiddle
passed
(lfs chroot) root:/sources/ninja-1.10.2# echo $?
0
```
TEXT
```
install -vm755 ninja /usr/bin/
```
```
install -vDm644 misc/bash-completion /usr/share/bash-completion/completions/ninja
```
```
install -vDm644 misc/zsh-completion  /usr/share/zsh/site-functions/_ninja
```
TEXT REMOVE
```
cd .. && rm -rf ninja-1.10.2

---

52
### 🔷 Meson-0.61.1
Распаковываем Meson-0.61.1 и переходим в папку с пакетом
```
tar xvf meson-0.61.1.tar.gz && cd meson-0.61.1
```
TEXT
```
python3 setup.py build
```
TEXT
```
python3 setup.py install --root=dest
```
```
cp -rv dest/* /
```
```
install -vDm644 data/shell-completions/bash/meson /usr/share/bash-completion/completions/meson
```
```
install -vDm644 data/shell-completions/zsh/_meson /usr/share/zsh/site-functions/_meson
```
TEXT REMOVE
```
cd .. && rm -rf meson-0.61.1

---

53
### 🔷 Coreutils-9.0
Распаковываем Coreutils-9.0 и переходим в папку с пакетом
```
tar xvf coreutils-9.0.tar.xz && cd coreutils-9.0
```
TEXT
```
patch -Np1 -i ../coreutils-9.0-i18n-1.patch
```
TEXT
```
autoreconf -fiv
```
```
FORCE_UNSAFE_CONFIGURE=1 ./configure --prefix=/usr --enable-no-install-program=kill,uptime
```
TEXT
```
time make -j8
```
```
real    0m5.864s
user    0m36.951s
sys     0m4.899s
(lfs chroot) root:/sources/coreutils-9.0# echo $?
0
```
TEXT
```
make NON_ROOT_USERNAME=tester check-root
```
```
============================================================================
Testsuite summary for GNU coreutils 9.0
============================================================================
# TOTAL: 33
# PASS:  20
# SKIP:  13
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
echo "dummy:x:102:tester" >> /etc/group
```
TEXT
```
chown -Rv tester . 
```
TEXT
```
su tester -c "PATH=$PATH make RUN_EXPENSIVE_TESTS=yes check"
```
TEXT
```
============================================================================
Testsuite summary for GNU coreutils 9.0
============================================================================
# TOTAL: 365
# PASS:  347
# SKIP:  18
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
sed -i '/dummy/d' /etc/group
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/coreutils-9.0# echo $?
0
```
TEXT
```
mv -v /usr/bin/chroot /usr/sbin
```
```
mv -v /usr/share/man/man1/chroot.1 /usr/share/man/man8/chroot.8
```
```
sed -i 's/"1"/"8"/' /usr/share/man/man8/chroot.8
```
TEXT REMOVE
```
cd .. && rm -rf coreutils-9.0

---

54
### 🔷 Check-0.15.2
Распаковываем Check-0.15.2 и переходим в папку с пакетом
```
tar xvf check-0.15.2.tar.gz && cd check-0.15.2
```
TEXT
```
./configure --prefix=/usr --disable-static
```
TEXT
```
time make -j8
```
```
real    0m1.313s
user    0m5.653s
sys     0m0.654s
(lfs chroot) root:/sources/check-0.15.2# echo $?
0
```
TEXT
```
make check
```
```
============================================================================
Testsuite summary for Check 0.15.2
============================================================================
# TOTAL: 9
# PASS:  9
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
make docdir=/usr/share/doc/check-0.15.2 install
```
```
(lfs chroot) root:/sources/check-0.15.2# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf check-0.15.2

---

55
### 🔷 Diffutils-3.8
Распаковываем Diffutils-3.8 и переходим в папку с пакетом
```
tar xvf diffutils-3.8.tar.xz && cd diffutils-3.8
```
TEXT
```
./configure --prefix=/usr
```
TEXT
```
time make -j8
```
```
real    0m1.291s
user    0m6.019s
sys     0m0.921s
(lfs chroot) root:/sources/diffutils-3.8# echo $?
0
```
TEXT
```
make check
```
```
============================================================================
Testsuite summary for GNU diffutils 3.8
============================================================================
# TOTAL: 203
# PASS:  187
# SKIP:  16
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/diffutils-3.8# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf diffutils-3.8

---

56
### 🔷 Gawk-5.1.1
Распаковываем Gawk-5.1.1 и переходим в папку с пакетом
```
tar xvf gawk-5.1.1.tar.xz && cd gawk-5.1.1
```
TEXT
```
sed -i 's/extras//' Makefile.in
```
TEXT
```
./configure --prefix=/usr
```
TEXT
```
time make -j8
```
```
(lfs chroot) root:/sources/gawk-5.1.1# echo $?
0
```
TEXT
```
make check
```
```
ALL TESTS PASSED
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/gawk-5.1.1# echo $?
0
```
TEXT
```
mkdir -pv /usr/share/doc/gawk-5.1.1
```
```
cp -v doc/{awkforai.txt,*.{eps,pdf,jpg}} /usr/share/doc/gawk-5.1.1
```
TEXT REMOVE
```
cd .. && rm -rf gawk-5.1.1

---

57
### 🔷 Findutils-4.9.0
Распаковываем Findutils-4.9.0 и переходим в папку с пакетом
```
tar xvf findutils-4.9.0.tar.xz && cd findutils-4.9.0
```
TEXT
```
case $(uname -m) in
    i?86)   TIME_T_32_BIT_OK=yes ./configure --prefix=/usr --localstatedir=/var/lib/locate ;;
    x86_64) ./configure --prefix=/usr --localstatedir=/var/lib/locate ;;
esac
```
TEXT
```
time make -j8
```
```
real    0m2.229s
user    0m7.976s
sys     0m1.292s
(lfs chroot) root:/sources/findutils-4.9.0# echo $?
0
```
TEXT
```
chown -Rv tester .
```
```
su tester -c "PATH=$PATH make check"
```
```
============================================================================
Testsuite summary for GNU findutils 4.9.0
============================================================================
# TOTAL: 17
# PASS:  15
# SKIP:  2
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
make install
```
```
(lfs chroot) root:/sources/findutils-4.9.0# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf findutils-4.9.0

---

58
### 🔷 Groff-1.22.4
Распаковываем Groff-1.22.4 и переходим в папку с пакетом
```
tar xvf groff-1.22.4.tar.gz && cd groff-1.22.4
```
TEXT
```
PAGE=A4 ./configure --prefix=/usr
```
TEXT
```
time make -j1
```
```
real    0m33.189s
user    0m31.509s
sys     0m2.349s
(lfs chroot) root:/sources/groff-1.22.4# echo $?
0
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/groff-1.22.4# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf groff-1.22.4

---

59
### 🔷 GRUB-2.06
Распаковываем GRUB-2.06 и переходим в папку с пакетом
```
tar xvf grub-2.06.tar.xz && cd grub-2.06
```
TEXT
```
./configure --prefix=/usr --sysconfdir=/etc --disable-efiemu --disable-werror
```
TEXT
```
time make -j8
```
```
real    0m11.452s
user    0m57.743s
sys     0m8.807s
(lfs chroot) root:/sources/grub-2.06# echo $?
0
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/grub-2.06# echo $?
0
```
```
mv -v /etc/bash_completion.d/grub /usr/share/bash-completion/completions
```
TEXT REMOVE
```
cd .. && rm -rf grub-2.06

---

60
### 🔷 Gzip-1.11
Распаковываем Gzip-1.11 и переходим в папку с пакетом
```
tar xvf gzip-1.11.tar.xz && cd gzip-1.11
```
TEXT
```
./configure --prefix=/usr
```
TEXT
```
time make -j8
```
```
real    0m0.787s
user    0m3.024s
sys     0m0.537s
(lfs chroot) root:/sources/gzip-1.11# echo $?
0
```
TEXT
```
make check
```
```
============================================================================
Testsuite summary for gzip 1.11
============================================================================
# TOTAL: 23
# PASS:  22
# SKIP:  0
# XFAIL: 0
# FAIL:  1
# XPASS: 0
# ERROR: 0
============================================================================
See tests/test-suite.log
Please report to bug-gzip@gnu.org
============================================================================
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/gzip-1.11# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf gzip-1.11

---

61
### 🔷 IPRoute2-5.16.0
Распаковываем IPRoute2-5.16.0 и переходим в папку с пакетом
```
tar xvf iproute2-5.16.0.tar.xz && cd iproute2-5.16.0
```
TEXT
```
sed -i /ARPD/d Makefile
```
```
rm -fv man/man8/arpd.8
```
TEXT
```
time make -j8
```
```
real    0m3.820s
user    0m17.750s
sys     0m2.088s
(lfs chroot) root:/sources/iproute2-5.16.0# echo $?
0
```
TEXT
```
make SBINDIR=/usr/sbin install
```
```
(lfs chroot) root:/sources/iproute2-5.16.0# echo $?
0
```
TEXT
```
mkdir -pv /usr/share/doc/iproute2-5.16.0
```
```
cp -v COPYING README* /usr/share/doc/iproute2-5.16.0
```
TEXT REMOVE
```
cd .. && rm -rf iproute2-5.16.0

---

62
### 🔷 Kbd-2.4.0
Распаковываем Kbd-2.4.0 и переходим в папку с пакетом
```
tar xvf kbd-2.4.0.tar.xz && cd kbd-2.4.0
```
TEXT
```
patch -Np1 -i ../kbd-2.4.0-backspace-1.patch
```
TEXT
```
sed -i '/RESIZECONS_PROGS=/s/yes/no/' configure
```
```
sed -i 's/resizecons.8 //' docs/man/man8/Makefile.in
```
TEXT
```
./configure --prefix=/usr --disable-vlock
```
TEXT
```
time make -j8
```
```
real    0m2.697s
user    0m9.652s
sys     0m1.673s
(lfs chroot) root:/sources/kbd-2.4.0# echo $?
0
```
TEXT
```
make check
```
```
## ------------- ##
## Test results. ##
## ------------- ##

36 tests were successful.
4 tests were skipped.
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/kbd-2.4.0# echo $?
0
```
TEXT
```
mkdir -pv /usr/share/doc/kbd-2.4.0
```
```
cp -R -v docs/doc/* /usr/share/doc/kbd-2.4.0
```
TEXT REMOVE
```
cd .. && rm -rf kbd-2.4.0

---

63
### 🔷 Libpipeline-1.5.5
Распаковываем Libpipeline-1.5.5 и переходим в папку с пакетом
```
tar xvf libpipeline-1.5.5.tar.gz && cd libpipeline-1.5.5
```
TEXT
```
./configure --prefix=/usr
```
TEXT
```
time make -j8
```
```
real    0m0.965s
user    0m2.187s
sys     0m0.348s
(lfs chroot) root:/sources/libpipeline-1.5.5# echo $?
0
```
TEXT
```
make check
```
```
============================================================================
Testsuite summary for libpipeline 1.5.5
============================================================================
# TOTAL: 7
# PASS:  7
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/libpipeline-1.5.5# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf libpipeline-1.5.5

---

64
### 🔷 Make-4.3
Распаковываем Make-4.3 и переходим в папку с пакетом
```
tar xvf make-4.3.tar.gz && cd make-4.3
```
TEXT
```
./configure --prefix=/usr
```
TEXT
```
time make -j8
```
```
real    0m0.807s
user    0m3.999s
sys     0m0.437s
(lfs chroot) root:/sources/make-4.3# echo $?
0
```
TEXT
```
make check
```
```
90 Tests in 125 Categories Complete ... No Failures :-)


======================================================================
 Regression PASSED: GNU Make 4.3 (x86_64-pc-linux-gnu) built with gcc 
======================================================================
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/make-4.3# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf make-4.3

---

65
### 🔷 Patch-2.7.6
Распаковываем Patch-2.7.6 и переходим в папку с пакетом
```
tar xvf patch-2.7.6.tar.xz && cd patch-2.7.6
```
TEXT
```
./configure --prefix=/usr
```
TEXT
```
time make -j8
```
```
real    0m1.145s
user    0m4.512s
sys     0m0.508s
```
TEXT
```
make check
```
```
============================================================================
Testsuite summary for GNU patch 2.7.6
============================================================================
# TOTAL: 44
# PASS:  41
# SKIP:  1
# XFAIL: 2
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/patch-2.7.6# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf patch-2.7.6

---

66
### 🔷 Tar-1.34
Распаковываем Tar-1.34 и переходим в папку с пакетом
```
tar xvf tar-1.34.tar.xz && cd tar-1.34
```
TEXT
```
FORCE_UNSAFE_CONFIGURE=1 ./configure --prefix=/usr
```
TEXT
```
time make -j8
```
```
real    0m3.697s
user    0m12.489s
sys     0m1.404s
(lfs chroot) root:/sources/tar-1.34# echo $?
0
```
TEXT
```
make check
```
```
## ------------- ##
## Test results. ##
## ------------- ##

ERROR: 218 tests were run,
1 failed unexpectedly.
20 tests were skipped.
## -------------------------- ##
## testsuite.log was created. ##
## -------------------------- ##
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/tar-1.34# echo $?
0
```
```
make -C doc install-html docdir=/usr/share/doc/tar-1.34
```
TEXT REMOVE
```
cd .. && rm -rf tar-1.34

---

67
### 🔷 Texinfo-6.8
Распаковываем Texinfo-6.8 и переходим в папку с пакетом
```
tar xvf texinfo-6.8.tar.xz && cd texinfo-6.8
```
TEXT
```
./configure --prefix=/usr
```
TEXT
```
sed -e 's/__attribute_nonnull__/__nonnull/' -i gnulib/lib/malloc/dynarray-skeleton.c
```
TEXT
```
time make -j8
```
```
real    0m5.120s
user    0m14.095s
sys     0m1.413s
(lfs chroot) root:/sources/texinfo-6.8# echo $?
0
```
TEXT
```
make check
```
```
============================================================================
Testsuite summary for GNU Texinfo 6.8
============================================================================
# TOTAL: 1
# PASS:  1
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/texinfo-6.8# echo $?
0
```
TEXT
```
make TEXMF=/usr/share/texmf install-tex
```
TEXT
```
pushd /usr/share/info
  rm -v dir
  for f in *
    do install-info $f dir 2>/dev/null
  done
popd
```
TEXT REMOVE
```
cd .. && rm -rf texinfo-6.8

---

68
### 🔷 Vim-8.2.4383
Распаковываем Vim-8.2.4383 и переходим в папку с пакетом
```
tar xvf vim-8.2.4383.tar.gz && cd vim-8.2.4383
```
TEXT
```
echo '#define SYS_VIMRC_FILE "/etc/vimrc"' >> src/feature.h
```
TEXT
```
./configure --prefix=/usr
```
TEXT
```
time make -j8
```
```
real    0m9.314s
user    1m4.330s
sys     0m3.426s
(lfs chroot) root:/sources/vim-8.2.4383# echo $?
0
```
TEXT
```
chown -Rv tester .
```
TEXT
```
su tester -c "LANG=en_US.UTF-8 make -j1 test" &> vim-test.log
```
```
-------------------------------
Executed:  4683 Tests
 Skipped:   107 Tests
  Failed:     0 Tests

ALL DONE
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/vim-8.2.4383# echo $?
0
```
TEXT
```
ln -sv vim /usr/bin/vi
```
```
for L in  /usr/share/man/{,*/}man1/vim.1; do
    ln -sv vim.1 $(dirname $L)/vi.1
done
```
TEXT
```
ln -sv ../vim/vim82/doc /usr/share/doc/vim-8.2.4383
```
TEXT
```
cat > /etc/vimrc << "EOF"
" Begin /etc/vimrc

" Ensure defaults are set before customizing settings, not after
source $VIMRUNTIME/defaults.vim
let skip_defaults_vim=1

set nocompatible
set backspace=2
set mouse=
syntax on
if (&term == "xterm") || (&term == "putty")
  set background=dark
endif

" End /etc/vimrc
EOF
```
TEXT
```
vim -c ':options'
```
TEXT REMOVE
```
cd .. && rm -rf vim-8.2.4383

---

69
### 🔷 Eudev-3.2.11
Распаковываем Eudev-3.2.11 и переходим в папку с пакетом
```
tar xvf eudev-3.2.11.tar.gz && cd eudev-3.2.11
```
TEXT
```
./configure --prefix=/usr --bindir=/usr/sbin --sysconfdir=/etc --enable-manpages --disable-static
```
TEXT
```
time make -j8
```
```
real    0m3.619s
user    0m10.847s
sys     0m1.600s
(lfs chroot) root:/sources/eudev-3.2.11# echo $?
0
```
TEXT
```
mkdir -pv /usr/lib/udev/rules.d
```
```
mkdr -pv /etc/udev/rules.d
```
TEXT
```
make check
```
```
============================================================================
Testsuite summary for eudev 3.2.11
============================================================================
# TOTAL: 2
# PASS:  2
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/eudev-3.2.11# echo $?
0
```
TEXT
```
tar -xvf ../udev-lfs-20171102.tar.xz
```
```
make -f udev-lfs-20171102/Makefile.lfs install
```
TEXT
```
udevadm hwdb --update
```
TEXT
TEXT REMOVE
```
cd .. && rm -rf eudev-3.2.11

---

70
### 🔷 Man-DB-2.10.1
Распаковываем Man-DB-2.10.1 и переходим в папку с пакетом
```
tar xvf man-db-2.10.1.tar.xz && cd man-db-2.10.1
```
TEXT
```
./configure --prefix=/usr --docdir=/usr/share/doc/man-db-2.10.1 --sysconfdir=/etc --disable-setuid --enable-cache-owner=bin --with-browser=/usr/bin/lynx --with-vgrind=/usr/bin/vgrind --with-grap=/usr/bin/grap --with-systemdtmpfilesdir= --with-systemdsystemunitdir=
```
TEXT
```
time make -j8
```
```
real    0m2.864s
user    0m12.683s
sys     0m1.879s
(lfs chroot) root:/sources/man-db-2.10.1# echo $?
0
```
TEXT
```
make check
```
```
============================================================================
Testsuite summary for man-db 2.10.1
============================================================================
# TOTAL: 12
# PASS:  12
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/man-db-2.10.1# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf man-db-2.10.1

---

71
### 🔷 Procps-ng-3.3.17
Распаковываем Procps-ng-3.3.17 и переходим в папку с пакетом
```
tar xvf procps-ng-3.3.17.tar.xz && cd procps-3.3.17
```
TEXT
```
./configure --prefix=/usr --docdir=/usr/share/doc/procps-ng-3.3.17 --disable-static --disable-kill
```
TEXT
```
time make -j8
```
```
(lfs chroot) root:/sources/procps-3.3.17# echo $?
0
```
TEXT
```
make check
```
```
============================================================================
Testsuite summary for procps-ng 3.3.17
============================================================================
# TOTAL: 1
# PASS:  1
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/procps-3.3.17# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf procps-3.3.17

---

72
### 🔷 Util-linux-2.37.4
Распаковываем Util-linux-2.37.4 и переходим в папку с пакетом
```
tar xvf util-linux-2.37.4.tar.xz && cd util-linux-2.37.4
```
TEXT
```
./configure ADJTIME_PATH=/var/lib/hwclock/adjtime --bindir=/usr/bin --libdir=/usr/lib --sbindir=/usr/sbin --docdir=/usr/share/doc/util-linux-2.37.4 --disable-chfn-chsh --disable-login --disable-nologin --disable-su --disable-setpriv --disable-runuser --disable-pylibmount --disable-static --without-python --without-systemd --without-systemdsystemunitdir
```
TEXT
```
time make -j8
```
```
real    0m9.891s
user    1m2.491s
sys     0m7.631s
(lfs chroot) root:/sources/util-linux-2.37.4# echo $?
0
```
TEXT
```
rm tests/ts/lsns/ioctl_ns
```
TEXT
```
chown -Rv tester .
```
```
su tester -c "make -k check"
```
```
---------------------------------------------------------------------
  All 212 tests PASSED
---------------------------------------------------------------------
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/util-linux-2.37.4# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf util-linux-2.37.4

---

73
### 🔷 E2fsprogs-1.46.5
Распаковываем E2fsprogs-1.46.5 и переходим в папку с пакетом
```
tar xvf e2fsprogs-1.46.5.tar.gz && cd e2fsprogs-1.46.5
```
TEXT
```
mkdir -v build && cd build
```
TEXT
```
../configure --prefix=/usr --sysconfdir=/etc --enable-elf-shlibs --disable-libblkid --disable-libuuid --disable-uuidd --disable-fsck
```
TEXT
```
time make -j8
```
```
real    0m6.797s
user    0m30.926s
sys     0m3.448s
(lfs chroot) root:/sources/e2fsprogs-1.46.5/build# echo $?
0
```
TEXT
```
make check
```
```
371 tests succeeded     0 tests failed
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/e2fsprogs-1.46.5/build# echo $?
0
```
TEXT
```
rm -fv /usr/lib/{libcom_err,libe2p,libext2fs,libss}.a
```
TEXT
```
gunzip -v /usr/share/info/libext2fs.info.gz
```
```
install-info --dir-file=/usr/share/info/dir /usr/share/info/libext2fs.info
```
TEXT
```
makeinfo -o doc/com_err.info ../lib/et/com_err.texinfo
```
```
install -v -m644 doc/com_err.info /usr/share/info
```
```
install-info --dir-file=/usr/share/info/dir /usr/share/info/com_err.info
```
TEXT REMOVE
```
cd .. && cd .. && rm -rf util-linux-2.37.4

---

74
### 🔷 Sysklogd-1.5.1
Распаковываем Sysklogd-1.5.1 и переходим в папку с пакетом
```
tar xvf sysklogd-1.5.1.tar.gz && cd sysklogd-1.5.1 
```
TEXT
```
sed -i '/Error loading kernel symbols/{n;n;d}' ksym_mod.c
```
```
sed -i 's/union wait/int/' syslogd.c
```
TEXT
```
time make -j8
```
```
real    0m0.306s
user    0m0.560s
sys     0m0.041s
(lfs chroot) root:/sources/sysklogd-1.5.1# echo $?
0
```
TEXT
```
make BINDIR=/sbin install
```
```
(lfs chroot) root:/sources/sysklogd-1.5.1# echo $?
0
```
TEXT
```
cat > /etc/syslog.conf << "EOF"
# Begin /etc/syslog.conf

auth,authpriv.* -/var/log/auth.log
*.*;auth,authpriv.none -/var/log/sys.log
daemon.* -/var/log/daemon.log
kern.* -/var/log/kern.log
mail.* -/var/log/mail.log
user.* -/var/log/user.log
*.emerg *

# End /etc/syslog.conf
EOF
```
TEXT REMOVE
```
cd .. && rm -rf sysklogd-1.5.1

---
75
### 🔷 Sysvinit-3.01
Распаковываем Sysvinit-3.01 и переходим в папку с пакетом
```
tar xvf sysvinit-3.01.tar.xz && cd sysvinit-3.01
```
TEXT
```
patch -Np1 -i ../sysvinit-3.01-consolidated-1.patch
```
TEXT
```
time make -j8
```
```
real    0m0.410s
user    0m1.198s
sys     0m0.106s
(lfs chroot) root:/sources/sysvinit-3.01# echo $?
0
```
TEXT
```
make install
```
```
(lfs chroot) root:/sources/sysvinit-3.01# echo $?
0
```
TEXT REMOVE
```
cd .. && rm -rf sysvinit-3.01

---

















