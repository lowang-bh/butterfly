
## 格式化磁盘

- LVM

```sh
pvcreate /dev/xvdb1
vgcreate VolGroupData /dev/xvdb1
lvcreate -l100%FREE -n Data VolGroupData
mkfs.xfs /dev/VolGroupData/Data
echo "/dev/VolGroupData/Data /data xfs defaults 0 0" >> /etc/fstab
mkdir /data
mount -a
```

- gdisk

```sh
# gdisk
parted -s /dev/sdb mklabel gpt
parted -s /dev/sdb mkpart primary 1 100%
mkfs.xfs /dev/sdb1
echo "/dev/sdb1   /data1  xfs defaults        0 0" >> /etc/fstab
mount -a
```

- lvm扩容

```sh
fdisk /dev/vdd
pvcreate /dev/vdd1
vgs
vgextend centos /dev/vdd1
lvdisplay
lvextend -l +100%FREE /dev/centos/root
xfs_growfs /dev/centos/root
# 如果是ext3,ext4, 用resize2fs扩容
resize2fs /dev/VolGroup/lv_root
```

- pv扩容

```
pvresize -v /dev/vdb
lvextend -l +100%FREE /dev/mapper/docker-docker
xfs_growfs /dev/mapper/docker-docker
```
