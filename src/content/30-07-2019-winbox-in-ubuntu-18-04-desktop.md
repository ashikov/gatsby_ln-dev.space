---
title: "Winbox в Ubuntu 18.04 Desktop"
date: "2019-07-30"
draft: false
path: "/blog/30-07-2019-winbox-in-ubuntu-18-04-desktop"
---

Эта небольшая заметка для тех, кто хочет использовать Winbox, для администрирования оборудования Mikrotik в своей уютной debian based операционной системе.

### Устанавливаем Wine

```shell
ln@dev:~$ sudo apt install wine -y
```

### Тянем Winbox с сайта Mikrotik

Актуальную ссылку можно получить [тут](https://mikrotik.com/download).

```shell
ln@dev:~$ wget https://download.mikrotik.com/routeros/winbox/3.19/winbox.exe
```

### Перебрасываем исполняемый файл в /bin

```shell
ln@dev:~$ sudo mv winbox.exe /bin
```

### Добавляем alias в свой .bashrc

Это нужно для запуска Winbox непосредственно из терминала по имени. Подробнее про назначение .bashrc можно почитать, например, [тут](https://unix.stackexchange.com/questions/129143/what-is-the-purpose-of-bashrc-and-how-does-it-work).

Добавить строку можно с помощью любого текстового редактора, но я считаю что для одной строки открывать его слишком долго.

```shell
ln@dev:~$ echo "alias winbox='wine /bin/winbox.exe'" >> ~/.bashrc
```

Аккуратней с __>>__, если заменить их на __>__, содержимое файла будет перезаписано. :) Делайте бекап перед тем как экспериментировать.

### Выполним скрипт

```shell
ln@dev:~$ source ~/.bashrc
```

### Проверим результат

```shell
ln@dev:~$ winbox
```

Должно открыться знакомое окно подключения к устройствам Mikrotik.
