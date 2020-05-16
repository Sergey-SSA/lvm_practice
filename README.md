*Работа с LVM*

_Подключил тестовый стенд [stands-03-lvm](https://gitlab.com/otus_linux/stands-03-lvm.git)_
`git clone https://gitlab.com/otus_linux/stands-03-lvm.git`

_Запустил тестовый стенд в директории stands-03-lvm_
`vagrant up`

_Подключился к нему_
`vagrant ssh lvm`

_Проверил диски_
`lsblk`
>NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
>sda                       8:0    0   40G  0 disk
>├─sda1                    8:1    0    1M  0 part
>├─sda2                    8:2    0    1G  0 part /boot
>└─sda3                    8:3    0   39G  0 part
>  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
>  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
>sdb                       8:16   0   10G  0 disk
>sdc                       8:32   0    2G  0 disk
>sdd                       8:48   0    1G  0 disk
>sde                       8:64   0    1G  0 disk`

_Проверил утилитой *lvmdiskscan*_
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

_Разметил диск для будующего использования LVM - *создал PV*_
`sudo pvcreate /dev/sdb`
>Physical volume "/dev/sdb" successfully created.`

_Далее создавал *первый уровень абстракции - VG*_
`sudo vgcreate otus /dev/sdb`
>Volume group "otus" successfully created

_Далее создал *Logical Volume - LV*_
`sudo lvcreate -l+80%FREE -n test otus`
>Logical volume "test" created.

_Посмотрел информацию о созданной VG_
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

_Посмотрел какие *диски входят в VG*_
`sudo vgdisplay -v otus | grep Na`
>  VG Name               otus
>  LV Name                test
>  VG Name                otus
>  PV Name               /dev/sdb

_Более подробная *информация по LV*_
` sudo lvdisplay /dev/otus/test
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

_В сжатом виде *информация по VG*_
`sudo vgs`
>  VG         `PV `LV `SN Attr   VSize   VFree
>  VolGroup00   1   2   0 wz--n- <38.97g    0
>  otus         1   1   0 wz--n- <10.00g 2.00g

_В сжатом виде *информация по LV*_
`sudo lvs`
>  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
>  LogVol00 VolGroup00 -wi-ao---- <37.47g
>  LogVol01 VolGroup00 -wi-ao----   1.50g
>  test     otus       -wi-a-----  <8.00g

_Из свободного места создал ещё один LV, не экстентами как в прошлый раз, а значением в мб._
`sudo lvcreate -L100M -n small otus`
>Logical volume "small" created.

_Проверил LV_
`sudo lvs`
>  LogVol00 VolGroup00 -wi-ao---- <37.47g
>  LogVol01 VolGroup00 -wi-ao----   1.50g
>  small    otus       -wi-a----- 100.00m
>  test     otus       -wi-a-----  <8.00g
*Выше видно что появился LV-small VG-otus на 100мб*

_Далее на LV test создал файловую систему_
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
` sudo mkdir /data

И смонтировал недавно созданный LV test в директорию data
` sudo mount /dev/otus/test /data/
mount: mount point /data/ does not exist

Проверил
` sudo mount |grep /data
/dev/mapper/otus-test on /data type ext4 (rw,relatime,seclabel,data=ordered)

Далее встала задача расширить место для /data т.к. 8 Гб не хватает. Для расширения VG буду использовать блочное утройство sdc, создав на нём PV
` sudo pvcreate /dev/sdc
Physical volume "/dev/sdc" successfully created.

Далее расширил VG otus
` sudo vgextend otus /dev/sdc
Volume group "otus" successfully extended

Проверил что диск присутствует в новой VG
` sudo vgdisplay -v otus | grep 'PV Na'
  PV Name               /dev/sdb
  PV Name               /dev/sdc

Проверил, добавилось ли место к 8 Гб
` sudo vgs
  VG         `PV `LV `SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g     0
  otus         2   2   0 wz--n-  11.99g <3.90g

Сымитировал занятое место на диске командой dd
` sudo dd if=/dev/zero of=/data/test.log bs=1M count=8000 status=progress
7941914624 bytes (7.9 GB) copied, 22.093150 s, 359 MB/s
dd: error writing ‘/data/test.log’: No space left on device
7880+0 records in
7879+0 records out
8262189056 bytes (8.3 GB) copied, 23.0594 s, 358 MB/s

Проверил что на диске занято 100%
` df -hT /data/
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4  7.8G  7.8G     0 100% /data

Увеличил LV, оставив 20% для снэпшотов
` sudo lvextend -l+80%FREE /dev/otus/test
Size of logical volume otus/test changed from <8.00 GiB (2047 extents) to <11.12 GiB (2846 extents).
  Logical volume otus/test successfully resized.

Сравнил: было test otus -wi-ao---- <8.00g
Стало
` lvs /dev/otus/test
test otus -wi-ao---- <11.12g
*Выше видно, что LV расширился с 8 Гб до 11.12Гб*

Но файловая система осталась прежней, это видно командой
` df -hT /data/
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4  7.8G  7.8G     0 100% /data

Далее выполнил resize файловой системы
` sudo resize2fs /dev/otus/test
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/otus/test is mounted on /data; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 2
The filesystem on /dev/otus/test is now 2914304 blocks long.

И проверил результат ` df -hT /data/
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4   11G  7.8G  2.6G  76% /data
*Выше видно что теперь занято не 100% как было ранее, а 76%. т.е. отработала утилита resize*

**Отработал ситуацию когда мне нужно уменьшить LV (забыл дать место для снэпшотов)**

Сначала отмонтировал
` sudo umount /data/

Затем проверил на ошибки
` sudo e2fsck -fy /dev/otus/test
e2fsck 1.42.9 (28-Dec-2013)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/otus/test: 12/729088 files (0.0% non-contiguous), 2105907/2914304 blocks

Опять выполнил resize2fs файловой системы
` sudo resize2fs /dev/otus/test 10G
resize2fs 1.42.9 (28-Dec-2013)
Resizing the filesystem on /dev/otus/test to 2621440 (4k) blocks.
The filesystem on /dev/otus/test is now 2621440 blocks long.

И уменьшил размер тома
` sudo lvreduce /dev/otus/test -L 10G
 WARNING: Reducing active logical volume to 10.00 GiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce otus/test? [y/n]: y
  Size of logical volume otus/test changed from <11.12 GiB (2846 extents) to 10.00 GiB (2560 extents).
  Logical volume otus/test successfully resized.

Смонтировал обратно
` sudo mount /dev/otus/test /data/

И проверил что файловая система стала нужного размера
` df -hT /data/
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4  9.8G  7.8G  1.6G  84% /data
*Выше видно, что размер стал меньше. Был 11Гб, стал 9.8Гб*

Проверил LV
` sudo lvs /dev/otus/test
  LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  test otus -wi-ao---- 10.00g

Далее проверил как работают снапшоты - команда lvcreate с флагом -s указывающий что нужно сделать снимок
` sudo lvcreate -L 500M -s -n test-snap /dev/otus/test
Logical volume "test-snap" created

Проверил с помошью VGS
` sudo vgs -o +lv_size,lv_name | grep test
 otus         2   3   1 wz--n-  11.99g <1.41g  10.00g test
 otus         2   3   1 wz--n-  11.99g <1.41g 500.00m test-snap

Анализировал вывод команды lsblk
` lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk
├─otus-small            253:3    0  100M  0 lvm
└─otus-test-real        253:4    0   10G  0 lvm
  ├─otus-test           253:2    0   10G  0 lvm  /data
  └─otus-test--snap     253:6    0   10G  0 lvm
sdc                       8:32   0    2G  0 disk
├─otus-test-real        253:4    0   10G  0 lvm ----- оригинальный LV
│ ├─otus-test           253:2    0   10G  0 lvm  /data
│ └─otus-test--snap     253:6    0   10G  0 lvm ----- снапшот
└─otus-test--snap-cow   253:5    0  500M  0 lvm ----- в него пишутся все изменения (Copy-on-Write)
  └─otus-test--snap     253:6    0   10G  0 lvm
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk

**Смонтировал снапшот как любой другой LV**

Создал для него директорию
` sudo mkdir /data-snap

` sudo mount /dev/otus/test-snap /data-snap/

` ll /data-snap/
total 8068564
drwx------. 2 root root      16384 May 14 17:26 lost+found
-rw-r--r--. 1 root root 8262189056 May 14 17:30 test.log

И отмонтировал
` sudo umount /data-snap

Проверил возможность откатиться на снапшот. Для наглядности сначала удалил логи
` rm /data/test.log

Проверил удаление
` ll
total 16
drwx------. 2 root root 16384 May 14 17:26 lost+found

Отмонтировал
` cd ..

` sudo umount /data

Откатил
` lvconvert --merge /dev/otus/test-snap
  Merging of volume otus/test-snap started.
  otus/test: Merged: 100.00%

` mount /dev/otus/test /data

` ll /data
drwx------. 2 root root      16384 May 14 17:26 lost+found
-rw-r--r--. 1 root root 8262189056 May 14 17:30 test.log
*Как видно выше, файл test.log восстановился из снапштота*

**Далее поработал с LVM Mirroring**
` pvcreate /dev/sd{d,e}
  Physical volume "/dev/sdd" successfully created.
  Physical volume "/dev/sde" successfully created.

` vgcreate vg0 /dev/sd{d,e}
Volume group "vg0" successfully created

` lvcreate -l+80%FREE -m1 -n mirror vg0
Logical volume "mirror" created

` lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  small    otus       -wi-a----- 100.00m
  test     otus       -wi-ao----  10.00g
  mirror   vg0        rwi-a-r--- 816.00m                                    100.00
*Выше видно LV mirror.*

**Выполнил задачу - Уменьшить том под / до 8G**

*Восстановил образ системы с начального стенда **Задача - Уменьшить том под / до 8G*

Подготовил временный том для раздела
` sudo pvcreate /dev/sdb
Physical volume "/dev/sdb" successfully created.

` sudo vgcreate vg_root /dev/sdb
Volume group "vg_root" successfully created

` sudo lvcreate -n lv_root -l +100%FREE /dev/vg_root
Logical volume "lv_root" created.

Создал на нём файловую систему и смонтировал его для переноса данных
` sudo mkfs.xfs /dev/vg_root/lv_root
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
` sudo mount /dev/vg_root/lv_root /mnt

Скопировал все данные из / в /mnt
` sudo yum install xfsdump

` chmod ugo+rwx mnt/

` sudo -i

` xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt

**Переконфигурировал grup, чтобы при старте перейти в новый /**
Сымитировал текущий root -` сделал в него chroot и обновил grub
` sudo -i

` for i in /proc/ /sys/ /dev/ /run/ /boot/
do mount --bind $i /mnt/$i
done

` chroot /mnt/

` grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done

Обновил образ initrd
` cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
*** Generating early-microcode cpio image contents ***
*** Constructing AuthenticAMD.bin ****
*** Store current command line parameters ***
*** Creating image file ***
*** Creating microcode section ***
*** Created microcode section ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***

*Чтобы при загрузке был смонтирован нужный root редактирую файл* /boot/grub2/grub.cfg заменил rd.lvm.lv=VolGroup00/LogVol00 на rd.lvm.lv=vg_root-lv_root
` reboot

После успешной перезагрузки проверил утилитой lsblk
` lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:2    0 37.5G  0 lvm
sdb                       8:16   0   10G  0 disk
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk

*Далее изменил размер старой VG и вернул на него /*
Для этого удалил старый LV на 40Гб и создал новый на 8Гб
`  lvremove /dev/VolGroup00/LogVol00
Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y
  Logical volume "LogVol00" successfully removed

Проверил удаление LV
` lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk

И создаю новый на 8Гб
`  lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/VolGroup00/LogVol00.
  Logical volume "LogVol00" created.

И ещё раз проверил
` lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:2    0    8G  0 lvm

Далее проделал на нём те же операции что и ранее в первый раз
` mkfs.xfs /dev/VolGroup00/LogVol00

` mount /dev/VolGroup00/LogVol00 /mnt

` xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
xfsdump: ending media file
xfsdump: media file size 694581152 bytes
xfsdump: dump size (non-dir files) : 681277296 bytes
xfsdump: dump complete: 15 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 15 seconds elapsed
xfsrestore: Restore Status: SUCCESS

Далее как и в первый раз переконфигурировал grub, за исключением правки /etc/grub2/grub.cfg
` for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done

` chroot /mnt/

` grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done

` cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;
s/.img//g"` --force; done

Выделил том под /var в зеркало, для Для него на свободном диске sdd создал зеркало
` pvcreate /dev/sdc /dev/sdd
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.

` vgcreate vg_var /dev/sdc /dev/sdd
  Volume group "vg_var" successfully created

` lvcreate -L 950M -m1 -n lv_var vg_var
  Rounding up size to full physical extent 952.00 MiB
  Logical volume "lv_var" created.

Проверил содание зеркала на диске sdd
` lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                        8:0    0   40G  0 disk
├─sda1                     8:1    0    1M  0 part
├─sda2                     8:2    0    1G  0 part /boot
└─sda3                     8:3    0   39G  0 part
  ├─VolGroup00-LogVol01  253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00  253:2    0    8G  0 lvm  /
sdb                        8:16   0   10G  0 disk
└─vg_root-lv_root        253:0    0   10G  0 lvm
sdc                        8:32   0    2G  0 disk
├─vg_var-lv_var_rmeta_0  253:3    0    4M  0 lvm
│ └─vg_var-lv_var        253:7    0  952M  0 lvm
└─vg_var-lv_var_rimage_0 253:4    0  952M  0 lvm
  └─vg_var-lv_var        253:7    0  952M  0 lvm
sdd                        8:48   0    1G  0 disk
├─vg_var-lv_var_rmeta_1  253:5    0    4M  0 lvm
│ └─vg_var-lv_var        253:7    0  952M  0 lvm
└─vg_var-lv_var_rimage_1 253:6    0  952M  0 lvm
  └─vg_var-lv_var        253:7    0  952M  0 lvm
sde                        8:64   0    1G  0 disk

Создал на нём файловую систему и переместил /var
` mkfs.ext4 /dev/vg_var/lv_var
Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

` mount /dev/vg_var/lv_var /mnt

` cp -aR /var/* /mnt/

` rsync -avHPSAX /var/ /mnt/
sending incremental file list
./
.updated
            163 100%    0.00kB/s    0:00:00 (xfr`1, ir-chk=1022/1024)

sent 130,798 bytes  received 565 bytes  262,726.00 bytes/sec
total size is 90,668,197  speedup is 690.21

На всякий случай сохранил старое содержимое var
` mkdir /tmp/oldvar && mv /var/* /tmp/oldvar

И смонтировал новый var в каталог /var
` umount /mnt

` mount /dev/vg_var/lv_var /var
Для автоматичского монтирования записал данные в fstab

` echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" `` /etc/fstab

И успешно перезагрзился в новый уже уменьшеный root...
Проверил
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                        8:0    0   40G  0 disk
├─sda1                     8:1    0    1M  0 part
├─sda2                     8:2    0    1G  0 part /boot
└─sda3                     8:3    0   39G  0 part
  ├─VolGroup00-LogVol00  253:0    0    8G  0 lvm  /
  └─VolGroup00-LogVol01  253:1    0  1.5G  0 lvm  [SWAP]
sdb                        8:16   0   10G  0 disk
└─vg_root-lv_root        253:7    0   10G  0 lvm
sdc                        8:32   0    2G  0 disk
├─vg_var-lv_var_rmeta_0  253:2    0    4M  0 lvm
│ └─vg_var-lv_var        253:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0 253:3    0  952M  0 lvm
  └─vg_var-lv_var        253:6    0  952M  0 lvm  /var
sdd                        8:48   0    1G  0 disk
├─vg_var-lv_var_rmeta_1  253:4    0    4M  0 lvm
│ └─vg_var-lv_var        253:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1 253:5    0  952M  0 lvm
  └─vg_var-lv_var        253:6    0  952M  0 lvm  /var
sde                        8:64   0    1G  0 disk

После перазагрузки удалил временную VG в порядке очерёдности команд:
` lvremove /dev/vg_root/lv_root
Do you really want to remove active logical volume vg_root/lv_root? [y/n]: y
  Logical volume "lv_root" successfully removed

` vgremove /dev/vg_root
  Volume group "vg_root" successfully removed

` pvremove /dev/sdb
 Labels on physical volume "/dev/sdb" successfully wiped.

Далее выделил том под /home по тому же принципу когда делал /var
` lvcreate -n LogVol_Home -L 2G /dev/VolGroup00
  Logical volume "LogVol_Home" created.

` mkfs.xfs /dev/VolGroup00/LogVol_Home
meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

` meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

` mount /dev/VolGroup00/LogVol_Home /mnt/

` cp -aR /home/* /mnt/

` rm -rf /home/*

` umount /mnt

` mount /dev/VolGroup00/LogVol_Home /home/

Для автоматического монтирования поправил /etc/fstab
` echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" `` /etc/fstab

И сделал том для снапшотов
Сгенерировал файл в /home/
` touch /home/file{1..20}

Снял снапшот
` lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home
  Rounding up size to full physical extent 128.00 MiB
  Logical volume "home_snap" created.

Удалил часть файлов
` rm -f /home/file{11..20}

Воспроизвёл процесс восстановления из снапшота
` umount /home

` lvconvert --merge /dev/VolGroup00/home_snap
  Merging of volume VolGroup00/home_snap started.
  VolGroup00/LogVol_Home: Merged: 100.00%

` mount /home и подготовил временный том для раздела__
` sudo pvcreate /dev/sdb
Physical volume "/dev/sdb" successfully created.

` sudo vgcreate vg_root /dev/sdb
Volume group "vg_root" successfully created

` sudo lvcreate -n lv_root -l +100%FREE /dev/vg_root
Logical volume "lv_root" created.

Создал на нём файловую систему и смонтировал его для переноса данных
` sudo mkfs.xfs /dev/vg_root/lv_root
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
` sudo mount /dev/vg_root/lv_root /mnt

Скопировал все данные из / в /mnt
` sudo yum install xfsdump

` chmod ugo+rwx mnt/

` sudo -i

` xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt

**Переконфигурировал grup, чтобы при старте перейти в новый /**
Сымитировал текущий root -` сделал в него chroot и обновил grub
` sudo -i

` for i in /proc/ /sys/ /dev/ /run/ /boot/
do mount --bind $i /mnt/$i
done

` chroot /mnt/

` grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done

Обновил образ initrd
` cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
*** Generating early-microcode cpio image contents ***
*** Constructing AuthenticAMD.bin ****
*** Store current command line parameters ***
*** Creating image file ***
*** Creating microcode section ***
*** Created microcode section ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***

*Чтобы при загрузке был смонтирован нужный root редактирую файл* /boot/grub2/grub.cfg заменил rd.lvm.lv=VolGroup00/LogVol00 на rd.lvm.lv=vg_root-lv_root
` reboot

После успешной перезагрузки проверил утилитой lsblk
` lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:2    0 37.5G  0 lvm
sdb                       8:16   0   10G  0 disk
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk

*Далее изменил размер старой VG и вернул на него /*
Для этого удалил старый LV на 40Гб и создал новый на 8Гб
`  lvremove /dev/VolGroup00/LogVol00
Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y
  Logical volume "LogVol00" successfully removed

Проверил удаление LV
` lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk

И создаю новый на 8Гб
`  lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/VolGroup00/LogVol00.
  Logical volume "LogVol00" created.

И ещё раз проверил
` lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:2    0    8G  0 lvm

Далее проделал на нём те же операции что и ранее в первый раз
` mkfs.xfs /dev/VolGroup00/LogVol00

` mount /dev/VolGroup00/LogVol00 /mnt

` xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
xfsdump: ending media file
xfsdump: media file size 694581152 bytes
xfsdump: dump size (non-dir files) : 681277296 bytes
xfsdump: dump complete: 15 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 15 seconds elapsed
xfsrestore: Restore Status: SUCCESS

Далее как и в первый раз переконфигурировал grub, за исключением правки /etc/grub2/grub.cfg
` for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done

` chroot /mnt/

` grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done

` cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;
s/.img//g"` --force; done

Выделил том под /var в зеркало, для Для него на свободном диске sdd создал зеркало
` pvcreate /dev/sdc /dev/sdd
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.

` vgcreate vg_var /dev/sdc /dev/sdd
  Volume group "vg_var" successfully created

` lvcreate -L 950M -m1 -n lv_var vg_var
  Rounding up size to full physical extent 952.00 MiB
  Logical volume "lv_var" created.

Проверил содание зеркала на диске sdd
` lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                        8:0    0   40G  0 disk
├─sda1                     8:1    0    1M  0 part
├─sda2                     8:2    0    1G  0 part /boot
└─sda3                     8:3    0   39G  0 part
  ├─VolGroup00-LogVol01  253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00  253:2    0    8G  0 lvm  /
sdb                        8:16   0   10G  0 disk
└─vg_root-lv_root        253:0    0   10G  0 lvm
sdc                        8:32   0    2G  0 disk
├─vg_var-lv_var_rmeta_0  253:3    0    4M  0 lvm
│ └─vg_var-lv_var        253:7    0  952M  0 lvm
└─vg_var-lv_var_rimage_0 253:4    0  952M  0 lvm
  └─vg_var-lv_var        253:7    0  952M  0 lvm
sdd                        8:48   0    1G  0 disk
├─vg_var-lv_var_rmeta_1  253:5    0    4M  0 lvm
│ └─vg_var-lv_var        253:7    0  952M  0 lvm
└─vg_var-lv_var_rimage_1 253:6    0  952M  0 lvm
  └─vg_var-lv_var        253:7    0  952M  0 lvm
sde                        8:64   0    1G  0 disk

Создал на нём файловую систему и переместил /var
` mkfs.ext4 /dev/vg_var/lv_var
Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

` mount /dev/vg_var/lv_var /mnt

` cp -aR /var/* /mnt/

` rsync -avHPSAX /var/ /mnt/
sending incremental file list
./
.updated
            163 100%    0.00kB/s    0:00:00 (xfr`1, ir-chk=1022/1024)

sent 130,798 bytes  received 565 bytes  262,726.00 bytes/sec
total size is 90,668,197  speedup is 690.21

На всякий случай сохранил старое содержимое var
` mkdir /tmp/oldvar && mv /var/* /tmp/oldvar

И смонтировал новый var в каталог /var
` umount /mnt

` mount /dev/vg_var/lv_var /var
Для автоматичского монтирования записал данные в fstab

` echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" `` /etc/fstab

И успешно перезагрзился в новый уже уменьшеный root...
Проверил
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                        8:0    0   40G  0 disk
├─sda1                     8:1    0    1M  0 part
├─sda2                     8:2    0    1G  0 part /boot
└─sda3                     8:3    0   39G  0 part
  ├─VolGroup00-LogVol00  253:0    0    8G  0 lvm  /
  └─VolGroup00-LogVol01  253:1    0  1.5G  0 lvm  [SWAP]
sdb                        8:16   0   10G  0 disk
└─vg_root-lv_root        253:7    0   10G  0 lvm
sdc                        8:32   0    2G  0 disk
├─vg_var-lv_var_rmeta_0  253:2    0    4M  0 lvm
│ └─vg_var-lv_var        253:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0 253:3    0  952M  0 lvm
  └─vg_var-lv_var        253:6    0  952M  0 lvm  /var
sdd                        8:48   0    1G  0 disk
├─vg_var-lv_var_rmeta_1  253:4    0    4M  0 lvm
│ └─vg_var-lv_var        253:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1 253:5    0  952M  0 lvm
  └─vg_var-lv_var        253:6    0  952M  0 lvm  /var
sde                        8:64   0    1G  0 disk

После перазагрузки удалил временную VG в порядке очерёдности команд:
` lvremove /dev/vg_root/lv_root
Do you really want to remove active logical volume vg_root/lv_root? [y/n]: y
  Logical volume "lv_root" successfully removed

` vgremove /dev/vg_root
  Volume group "vg_root" successfully removed

` pvremove /dev/sdb
 Labels on physical volume "/dev/sdb" successfully wiped.

Далее выделил том под /home по тому же принципу когда делал /var
` lvcreate -n LogVol_Home -L 2G /dev/VolGroup00
  Logical volume "LogVol_Home" created.

` mkfs.xfs /dev/VolGroup00/LogVol_Home
meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

` meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

` mount /dev/VolGroup00/LogVol_Home /mnt/

` cp -aR /home/* /mnt/

` rm -rf /home/*

` umount /mnt

` mount /dev/VolGroup00/LogVol_Home /home/

Для автоматического монтирования поправил /etc/fstab
` echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" `` /etc/fstab

И сделал том для снапшотов
Сгенерировал файл в /home/
` touch /home/file{1..20}

Снял снапшот
` lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home
  Rounding up size to full physical extent 128.00 MiB
  Logical volume "home_snap" created.

Удалил часть файлов
` rm -f /home/file{11..20}

Воспроизвёл процесс восстановления из снапшота
` umount /home

` lvconvert --merge /dev/VolGroup00/home_snap
  Merging of volume VolGroup00/home_snap started.
  VolGroup00/LogVol_Home: Merged: 100.00%

` mount /home
