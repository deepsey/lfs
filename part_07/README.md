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
