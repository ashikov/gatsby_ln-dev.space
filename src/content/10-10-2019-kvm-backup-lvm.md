---
title: "KVM: Бекап VM дисковая подсистема которых основана на LVM-структуре"
date: "2019-10-10"
draft: false
path: "/blog/10-10-2019-kvm-backup-lvm"
---

В данной статье я хочу описать процесс создания резервной копии виртуальной машины, которая для хранения данных использует диск входящий в группу [__LVM__](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)).

Главное преимущество __Logical Volume Manager__, в данном случае, поддержка моментальных снимков состояния (__snapshot__) любого тома.

> Бекап снимается без непосредственного отключения виртуальной машины. Происходит сохранение данных состояния оперативной памяти, после чего активируется снапшот и __VM__ восстанавливает свою работу. Данный подход является оптимальным в случаях когда даже кратковременное отключение гостевой системы стоит очень дорого.

Я не буду приводить здесь настройку __LVM__, установку необходимых для [__KVM__](https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine) пакетов и то как все это связать вместе. Этот вопрос, возможно, будет рассмотрен в следующих постах на на тему виртуализации.

В качестве хост-системы выступает [__Ubuntu Server 18.04.3 LTS__](https://ubuntu.com/download/server), если кому-то важно знать эту информацию. :)

Взаимодействовать с виртуальными машинами мы будем через один из самых популярных CLI-интерфейсов - __virsh__.

---

В резервной копии будем хранить:

- Данные о размере диска
- Данные о пуле в котором находится диск
- Путь к __LVM__ тому
- __XML__ конфигурацию виртуально машины
- Данные состояния виртуальной машины на момент создания бекапа

---

Для начала, переключимся на root-a.

```shell
ln@dev:~$ sudo -i
```

Чтобы определить какие __LVM__ тома нужно резервировать, выведем детальную информацию о подключенных к __VM__ устройствах.

```shell
root@serv-ub:~$ virsh domblklist kvmserv --details
Type       Device     Target     Source
------------------------------------------------
block      disk       sda        /dev/vg-libvirt/kvmserv-lvm-disk-1
file       cdrom      hdb        -
```

Первым в списке мы как раз видим жесткий диск и его источник. Теперь нам необходимо получить информацию о этом __LVM__ разделе. Также мы указываем, что при выводе информации о размере раздела нужно использовать байты (__--units B__).

```shell
root@serv-ub:~$ lvdisplay /dev/vg-libvirt/kvmserv-lvm-disk-1 --units B
  --- Logical volume ---
  LV Path                /dev/vg-libvirt/kvmserv-lvm-disk-1
  LV Name                kvmserv-lvm-disk-1
  VG Name                vg-libvirt
  LV UUID                iF5uET-RfTI-w4uv-A1or-9mCg-Ncbz-Yr4ftz
  LV Write Access        read/write
  LV Creation host, time serv-ub, 2019-10-07 06:56:23 +0000
  LV Status              available
  # open                 2
  LV Size                268435456000 B
  Current LE             64000
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

Выведем детальную информацию о блочном устройстве VM.

```shell
root@serv-ub:~$ virsh domblkinfo kvmserv /dev/vg-libvirt/kvmserv-lvm-disk-1
Capacity:       268435456000
Allocation:     164283592704
Physical:       268435456000
```

Как видите __Capasity__ совпадает с данными которые мы получили ранее с помощью __lvdisplay__.

Запишем информацию о диске в файл бекапа.

```shell
root@serv-ub:~$ virsh domblkinfo kvmserv /dev/vg-libvirt/kvmserv-lvm-disk-1 > /mnt/backups/kvmserv-lvm-disk1
```

Дописываем в тот же файл информацию о пуле в котором находится данное устройство.

```shell
root@serv-ub:~$ virsh vol-pool /dev/vg-libvirt/kvmserv-lvm-disk-1 >> /mnt/backups/kvmserv-lvm-disk1
```

Туда же отправляем и путь к __LVM__ тому.

```shell
root@serv-ub:~$ echo "/dev/vg-libvirt/kvmserv-lvm-disk-1" >> /mnt/backups/kvmserv-lvm-disk1
```

Сохраняем __XML__ конфигурацию витруальной машины (просматривать её можно, например, пробросив [__STDOUT__](https://en.wikipedia.org/wiki/Standard_streams#Standard_output_(stdout)) через [__pipeline__](https://en.wikipedia.org/wiki/Pipeline_(Unix)) в [__less__](http://man7.org/linux/man-pages/man1/less.1.html)).

```shell
root@serv-ub:~$ virsh dumpxml kvmserv > /mnt/backups/kvmserv-lvm-disk1.xml
```

Сохраняем состояние оперативной памяти. Опция __--running__ говорит о том, что после восстановления виртуальная машина сразу будет в состоянии __running__.

> С этого момента __VM__ уходит в статус __shutoff__.

```shell
root@serv-ub:~$ virsh save kvmserv /mnt/backups/kvmserv.vmstate --running

Domain kvmserv saved to /mnt/backups/kvmserv.vmstate
```

Пока машина заморожена создаем snapshot __LVM__ тома.

> Размер __[COW-table](https://en.wikipedia.org/wiki/Copy-on-write) size__, который указывается в параметре __-L__ необходимо подбирать таким образом, чтобы значение __Allocated to snapshot__ (которое можно посмотреть, например, такой командой: __lvdisplay | grep "Allocated to snapshot"__) за время пока будет происходить архивация снепшота __НЕ__ достигло 100%. Если такое случится, то снепшот считается разрушенным и восстановиться из него не получится.

```shell
root@serv-ub:~$ lvcreate -s -n kvmserv-lvm-disk-1_snap -L16G /dev/vg-libvirt/kvmserv-lvm-disk-1
  Using default stripesize 64.00 KiB.
  Logical volume "kvmserv-lvm-disk-1_snap" created.
```

Восстанавливаем __VM__ из сохраненного ранее состояния.

> С этого момента виртуальная машина продолжает работать, дополнительно запускать её не нужно.

```shell
root@serv-ub:~$ virsh restore /mnt/backups/kvmserv.vmstate
Domain restored from /mnt/backups/kvmserv.vmstate
```

Сохраняем __snapshot__ пропуская его через архиватор.

```shell
root@serv-ub:~$ dd if=/dev/vg-libvirt/kvmserv-lvm-disk-1_snap | gzip -ck -3 > /mnt/backups/kvmserv-lvm-disk-1.gz
524288000+0 records in
524288000+0 records out
268435456000 bytes (268 GB, 250 GiB) copied, 2262.59 s, 119 MB/s
```

Степерь сжатия выбирайте на свое усмотрение и не забывайте, что без __-k__ __gzip__ удаляет исходный файл.

После архивации удаляем снапшот.

```shell
root@serv-ub:~$ lvremove /dev/vg-libvirt/kvmserv-lvm-disk-1_snap
Do you really want to remove and DISCARD active logical volume vg-libvirt/kvmserv-lvm-disk-1_snap? [y/n]: y
  Logical volume "kvmserv-lvm-disk-1_snap" successfully removed
```

В итоге у нас получился следующий набор файлов.

```shell
ln@serv-ub:~$ ls -l /mnt/backups/
total 40847552
-rw-r--r-- 1 root root 28974686049 Oct 10 04:01 kvmserv-lvm-disk-1.gz
-rw-r--r-- 1 root root         137 Oct 10 02:39 kvmserv-lvm-disk1
-rw-r--r-- 1 root root        5715 Oct 10 02:41 kvmserv-lvm-disk1.xml
-rw------- 1 root root 12853167635 Oct 10 03:05 kvmserv.vmstate
```

На этом создание резервной копии можно считать завершенным. Полученные данные позволят развернуть бекап даже в случае потери всей информации о виртуальной машине и её конфигурации.
