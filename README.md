# Reduce LVM Partition on CentOS 7

Sometimes when creating a new CentOS 7 server, the drive is partioned with the root, boot and swap, and then all the rest of the space is given to the home directory.

Here, we are going to reduce the size of the `/home` partition and allocate the remaining space back to the root partition.

## List Block Devices

List the current block devices

```
lsblk
```

It should look like this

```
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0  1.1T  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0  1.1T  0 part
  ├─centos-root 253:0    0   50G  0 lvm  /
  ├─centos-swap 253:1    0    4G  0 lvm  [SWAP]
  └─centos-home 253:2    0    1T  0 lvm  /home
sr0              11:0    1 1024M  0 rom
```

## Backup `/home`

Backup the contents of the `/home` directory

```
tar -czvf /root/home.tgz -C /home .
```

Test the backup

```
tar -tvf /root/home.tgz
```

## Reduce the Size of the `/home` Partition

Unmount home

```
umount /dev/mapper/centos-home
```

Remove the home logical volume

```
lvremove /dev/mapper/centos-home
```

Recreate a new 100GB logical volume for `/home`

```
lvcreate -L 100GB -n home centos
```

Format it

```
mkfs.xfs /dev/centos/home
```

Mount it

```
mount /dev/mapper/centos-home
```

## Extend `/root` volume

Extend the `/root` volume with all of the remaining space and resize the filesystem

```
lvextend -r -l +100%FREE /dev/mapper/centos-root
```

## Restore `/home`

Restore the contents of the `/home` directory

```
tar -xzvf /root/home.tgz -C /home
```

## List Block Devices

List the block devices again to make sure it was updated

```
lsblk
```

It should look like this

```
NAME            MAJ:MIN RM    SIZE RO TYPE MOUNTPOINT
sda               8:0    0    1.1T  0 disk
├─sda1            8:1    0      1G  0 part /boot
└─sda2            8:2    0    1.1T  0 part
  ├─centos-root 253:0    0 1012.8G  0 lvm  /
  ├─centos-swap 253:1    0      4G  0 lvm  [SWAP]
  └─centos-home 253:2    0    100G  0 lvm  /home
sr0              11:0    1   1024M  0 rom
```
