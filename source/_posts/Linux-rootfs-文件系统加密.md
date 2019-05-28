---
title: Linux rootfs 文件系统加密
date: 2019-05-12 21:32:43
tags: [Linux, Filesystem, Cryptsetup]
---

##  Cryptsetup 的使用

1. 首先是创建加密 key, 和待加密的虚拟块设备文件（也可以直接使用物理块设备测试） 。

```bash
➜ dd if=/dev/urandom of=key.img bs=1024 count=1
➜ dd if=/dev/zero of=data.img bs=1 count seek=1G count=0
```

2. 接着开始使用 **cryptsetup** 格式化加密设备，这里使用 **luks** 加密格式。

```bash
➜ sudo cryptsetup luksFormat data.img key.img

WARNING!
========
This will overwrite data on data.img irrevocably.

Are you sure? (Type uppercase yes): YES
```
> 注意这里 YES 是大写。
> 当然如果不指定 key 文件，它会提示你手动输入密码
> 批处理的话可以直接 `echo 'password' | cryptsetup luksFormat data.img`

格式化完后，可以看到 data.img 文件的 ***file*** 信息:

```
data.img: LUKS encrypted file, ver 1 [aes, xts-plain64, sha1] UUID: ec445173-d874-4d9b-953e-895114ecc4be
```

3. 打开 data.img 设备文件

   ```
   ➜ sudo cryptsetup luksOpen data.img rootfs --key-file key.img
   ```

   如果成功执行，可以在 lsblk 中看到我们的加密文件已经映射成功。

   ```
   ➜ sudo lsblk
   
   NAME                  MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
   loop1                   7:1    0     1G  0 loop
   └─rootfs              253:9    0  1022M  0 crypt
   vda                   252:0    0   120G  0 disk
   |-vda1                252:1    0  1023M  0 part  /boot
   |-vda2                252:2    0     1G  0 part
   `-vda3                252:3    0   118G  0 part
     |-hdd-root01        253:0    0     5G  0 lvm   /
     |-hdd-root02        253:1    0     5G  0 lvm
     `-hdd-totaldata     253:2    0   107G  0 lvm   /totaldata
   
   ➜ sudo losetup -l                                                           
   NAME       SIZELIMIT OFFSET AUTOCLEAR RO BACK-FILE
   /dev/loop1         0      0         1  0 /data/images/data.img
   ```

接下来我们就可以像操作块设备一样来操作它了。

比如,  我们首先创建一个文件系统。

```
➜ sudo mkfs.ext4 /dev/mapper/rootfs
mke2fs 1.42.13 (17-May-2015)
Creating filesystem with 261632 4k blocks and 65408 inodes
Filesystem UUID: d8ac8e88-bc85-42df-8bcd-24916dcf1bce
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```

4. 最后关闭加密设备

   ```
   ➜ sudo cryptsetup close rootfs
   ```

## 使用 Cryptsetup 加密系统

Linux 磁盘加密一般分为 home 目录加密码和 rootfs 加密， 以及全盘加密。

不同 linux 分行版上相关配置会略有不同， 主要体现在 grub 启动参数， 以及 initrd 上的配置。

这里主要讨论 ***Ubuntu 18.04*** rootfs 加密。

配置 initramfs

添加配置文件 /etc/initramfs-tools/cond.d/cryptsetup:

```
CRYPTESETUP=y
```

同时在 [crypttab](https://wiki.archlinux.org/index.php/Crypttab) and [fstab](https://wiki.archlinux.org/index.php/Fstab) 配置启动挂载信息:

```
/etc/crypttab
rootfs UUID=1c6509d8-d807-45a3-82ac-f39235298fef none luks

/etc/fstab
/dev/mapper/rootfs        /   ext4        defaults        0       2
```
最后生成 initramfs
```
➜ sudo update-initramfs -u
```

Ubuntu 18.04 中 grub 配置示例:
```
root=/dev/mapper/rootfs cryptopts=target=rootfs,source=UUID=1c6509d8-d807-45a3-82ac-f39235298fef,lvm=hdd-root02
```
CentOS 7 中 grub 配置示例:
```
root=/dev/mapper/luks-ae2ad428-3cd7-4fa0-9fa8-1a1c8b90b25a ro crashkernel=auto rd.luks.uuid=luks-ae2ad428-3cd7-4fa0-9fa8-1a1c8b90b25a rd.lvm.lv=ssd/root
```

> 配置 rootfs 加密， 必须更新 initramfs， 否则系统无法启动。
>
> 同时配置无密确解锁时，必须将 key 文件 注入 initrd 中。
> 
> 具体 **cryptopts** 参数可以查看 源文件：/usr/share/initramfs-tools/hooks/cryptroot:get_device_opts
