# DZ03lvm
На виртуальной машине с Ubuntu 24.04 и LVM.
1. Уменьшить том под / до 8G.
2. Выделить том под /home.
3. Выделить том под /var - сделать в mirror.
4. /home - сделать том для снапшотов.
5. Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).
6. Работа со снапшотами:
* сгенерить файлы в /home/;
* снять снапшот;
* удалить часть файлов;
* восстановится со снапшота.
7. На дисках попробовать поставить btrfs/zfs — с кэшем, снапшотами и разметить там каталог /opt.
```
gor@testsrv:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0 50,9M  1 loop /snap/snapd/25577
loop1                       7:1    0 63,8M  1 loop /snap/core20/2682
loop2                       7:2    0 73,9M  1 loop /snap/core22/2139
sda                         8:0    0   16G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1,8G  0 part /boot
└─sda3                      8:3    0 14,2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0   10G  0 disk
sdc                         8:32   0    2G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sr0                        11:0    1  3,1G  0 rom

root@testsrv:/home/gor# pvcreate /dev/sdb /dev/sdc /dev/sdd /dev/sde
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
  Physical volume "/dev/sde" successfully created.

root@testsrv:/home/gor# vgcreate vg_var /dev/sdc /dev/sdd
  Volume group "vg_var" successfully created
root@testsrv:/home/gor# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg   1   1   0 wz--n- <14,25g <4,25g
  vg_var      2   0   0 wz--n-   2,99g  2,99g

root@testsrv:/home/gor# lvcreate -L 900M -m1 -n lv_var vg_var
  Logical volume "lv_var" created.
root@testsrv:/home/gor# lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-vg -wi-ao----  10,00g
  lv_var    vg_var    rwi-a-r--- 900,00m                                    100,00

root@testsrv:/home/gor# mkfs.ext4 /dev/vg_var/lv_var
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 230400 4k blocks and 57600 inodes
Filesystem UUID: 0dce8ce1-a19c-48ab-ab81-11a76b9d1718
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

root@testsrv:/home/gor# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0 50,9M  1 loop /snap/snapd/25577
loop1                       7:1    0 63,8M  1 loop /snap/core20/2682
loop2                       7:2    0 73,9M  1 loop /snap/core22/2139
sda                         8:0    0   16G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1,8G  0 part /boot
└─sda3                      8:3    0 14,2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0   10G  0 disk
sdc                         8:32   0    2G  0 disk
├─vg_var-lv_var_rmeta_0   252:1    0    4M  0 lvm
│ └─vg_var-lv_var         252:5    0  900M  0 lvm
└─vg_var-lv_var_rimage_0  252:2    0  900M  0 lvm
  └─vg_var-lv_var         252:5    0  900M  0 lvm
sdd                         8:48   0    1G  0 disk
├─vg_var-lv_var_rmeta_1   252:3    0    4M  0 lvm
│ └─vg_var-lv_var         252:5    0  900M  0 lvm
└─vg_var-lv_var_rimage_1  252:4    0  900M  0 lvm
  └─vg_var-lv_var         252:5    0  900M  0 lvm
sde                         8:64   0    1G  0 disk
sr0                        11:0    1  3,1G  0 rom

root@testsrv:~# mount /dev/vg_var/lv_var /mnt/var_test/
root@testsrv:~# df
Filesystem                        1K-blocks    Used Available Use% Mounted on
tmpfs                                401032    1200    399832   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  10218772 6312320   3365780  66% /
tmpfs                               2005156       0   2005156   0% /dev/shm
tmpfs                                  5120       0      5120   0% /run/lock
/dev/sda2                           1768056  112116   1547808   7% /boot
tmpfs                                401028      16    401012   1% /run/user/100                                                                                                                                                             0
/dev/mapper/vg_var-lv_var            888472      24    825984   1% /mnt/var_test

root@testsrv:~# touch /mnt/var_test/file{1..20}


