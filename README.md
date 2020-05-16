**Работа с LVM**

Подключил тестовый стенд [stands-03-lvm](https://gitlab.com/otus_linux/stands-03-lvm.git)

`git clone https://gitlab.com/otus_linux/stands-03-lvm.git`

Запустил тестовый стенд в директории stands-03-lvm

`vagrant up`

Подключился к нему

`vagrant ssh lvm`

Проверил диски

`lsblk`

```
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk`
```

Проверил утилитой **lvmdiskscan**

`sudo lvmdiskscan`

> /dev/VolGroup00/LogVol00 [     <37.47 GiB]
>  /dev/VolGroup00/LogVol01 [       1.50 GiB]
>  /dev/sda2                [       1.00 GiB]
>  /dev/sda3                [     <39.00 GiB] LVM physical volume
>  /dev/sdb                 [      10.00 GiB]
>  /dev/sdc                 [       2.00 GiB]
>  /dev/sdd                 [       1.00 GiB]
>  /dev/sde                 [       1.00 GiB]
>  4 disks
>  3 partitions
>  0 LVM physical volume whole disks
>  1 LVM physical volume`

Разметил диск для будующего использования LVM - **создал PV**

`sudo pvcreate /dev/sdb`

>Physical volume "/dev/sdb" successfully created.`

Далее создавал **первый уровень абстракции - VG**

`sudo vgcreate otus /dev/sdb`

>Volume group "otus" successfully created

Далее создал **Logical Volume - LV**

`sudo lvcreate -l+80%FREE -n test otus`

>Logical volume "test" created.

Посмотрел **информацию** о созданной VG

`sudo vgdisplay`

> --- Volume group ---
>  VG Name               VolGroup00
>  System ID
>  Format                lvm2
>  Metadata Areas        1
>  Metadata Sequence No  3
>  VG Access             read/write
>  VG Status             resizable
>  MAX LV                0
>  Cur LV                2
>  Open LV               2
>  Max PV                0
>  Cur PV                1
>  Act PV                1
>  VG Size               <38.97 GiB
>  PE Size               32.00 MiB
>  Total PE              1247
>  Alloc PE / Size       1247 / <38.97 GiB
>  Free  PE / Size       0 / 0
>  VG UUID               SA8LTU-F2yz-FEV1-RdgT-hw0Z-iRxh-yHFKuU
>
>  --- Volume group ---
>  VG Name               otus
>  System ID
>  Format                lvm2
>  Metadata Areas        1
>  Metadata Sequence No  2
>  VG Access             read/write
>  VG Status             resizable
>  MAX LV                0
>  Cur LV                1
>  Open LV               0
>  Max PV                0
>  Cur PV                1
>  Act PV                1
>  VG Size               <10.00 GiB
>  PE Size               4.00 MiB
>  Total PE              2559
>  Alloc PE / Size       2047 / <8.00 GiB
>  Free  PE / Size       512 / 2.00 GiB
>  VG UUID               AXvOU1-13eM-4iat-rAtP-RWXM-ibMC-PlCwf2`

Посмотрел **какие диски входят в VG**

`sudo vgdisplay -v otus | grep Na`

>  VG Name               otus
>  LV Name                test
>  VG Name                otus
>  PV Name               /dev/sdb

Более подробная **информация по LV**

`sudo lvdisplay /dev/otus/test`

> --- Logical volume ---
>  LV Path                /dev/otus/test
>  LV Name                test
>  VG Name                otus
>  LV UUID                7SDqzF-Y4kl-UQ1Q-oc9j-HmTC-CWAC-WQBJPB
>  LV Write Access        read/write
>  LV Creation host, time lvm, 2020-05-13 11:45:57 +0000
>  LV Status              available
>  ` open                 0
>  LV Size                <8.00 GiB
>  Current LE             2047
>  Segments               1
>  Allocation             inherit
>  Read ahead sectors     auto
>  - currently set to     8192
>  Block device           253:2

В сжатом виде **информация по VG**

`sudo vgs`

>  VG         `PV `LV `SN Attr   VSize   VFree
>  VolGroup00   1   2   0 wz--n- <38.97g    0
>  otus         1   1   0 wz--n- <10.00g 2.00g

В сжатом виде **информация по LV**

`sudo lvs`

>  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
>  LogVol00 VolGroup00 -wi-ao---- <37.47g
>  LogVol01 VolGroup00 -wi-ao----   1.50g
>  test     otus       -wi-a-----  <8.00g

Из свободного места **создал ещё один LV, не экстентами как в прошлый раз, а значением в мб.**

`sudo lvcreate -L100M -n small otus`

>Logical volume "small" created.

Проверил LV

`sudo lvs`

>  LogVol00 VolGroup00 -wi-ao---- <37.47g
>  LogVol01 VolGroup00 -wi-ao----   1.50g
>  small    otus       -wi-a----- 100.00m
>  test     otus       -wi-a-----  <8.00g

_Выше видно что появился LV-small VG-otus на 100мб_

Далее на LV test создал файловую систему

`sudo mkfs.ext4 /dev/otus/test`

>mke2fs 1.42.9 (28-Dec-2013)
>Filesystem label=
>OS type: Linux
>Block size=4096 (log=2)
>Fragment size=4096 (log=2)
>Stride=0 blocks, Stripe width=0 blocks
>524288 inodes, 2096128 blocks
>104806 blocks (5.00%) reserved for the super user
>First data block=0
>Maximum filesystem blocks=2147483648
>64 block groups
>32768 blocks per group, 32768 fragments per group
>8192 inodes per group
>Superblock backups stored on blocks:
>        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632
>
>Allocating group tables: done
>Writing inode tables: done
>Creating journal (32768 blocks): done
>Writing superblocks and filesystem accounting information: done

Создал директорию для данных

`sudo mkdir /data`

И смонтировал недавно созданный LV test в директорию data

`sudo mount /dev/otus/test /data/`

>mount: mount point /data/ does not exist

Проверил

`sudo mount |grep /data`

>/dev/mapper/otus-test on /data type ext4 (rw,relatime,seclabel,data=ordered)

Далее встала задача **расширить место** для /data т.к. 8 Гб не хватает. Для расширения VG буду использовать блочное утройство sdc, создав на нём PV
