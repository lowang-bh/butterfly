# kubernetes

## 节点设置

### 关闭swap

```sh
swapoff -a
sed  -i '/swap/s/^/#/'  /etc/fstab
# ubuntu比较特殊，除了fstab，systemd也管理swap
systemctl status dev-sda2.swap
systemctl mask dev-sda2.swap --now
```

如果想删除swap分区，需要相应的修改grub，否则启动的时候找不到swap分区导致启动失败

### 关闭防火墙和selinux

```sh
systemctl disable firewalld

setenfore 0

sed -i '/^SELINUX=/cSELINUX=disabled' /etc/selinux/config
```

### 确保DNS存在

```sh
[root@kvm-k8s-master ~]# cat /etc/resolv.conf
nameserver 10.106.170.107
nameserver 10.106.170.108
```

### 添加内网镜像地址

```sh
echo "10.131.0.204 mirrors.bdp.idc" >> /etc/hosts
```

### 为docker添加磁盘

```sh
pvcreate /dev/vdb
vgcreate docker /dev/vdb
lvcreate -l100%FREE -n docker docker

mkdir -p /var/lib/docker
mkfs.xfs /dev/docker/docker -n ftype=1
echo "/dev/docker/docker      /var/lib/docker         xfs     defaults        0 0" >> /etc/fstab
mount -a
xfs_info /dev/docker/docker
```

### 全部完整commands

```sh
swapoff -a
sed  -i '/swap/s/^/#/'  /etc/fstab

systemctl disable firewalld
setenfore 0
sed -i '/^SELINUX=/cSELINUX=disabled' /etc/selinux/config

echo "10.131.0.204 mirrors.bdp.idc" >> /etc/hosts

pvcreate /dev/vdb
vgcreate docker /dev/vdb
lvcreate -l100%FREE -n docker docker

mkdir -p /var/lib/docker
mkfs.xfs /dev/docker/docker -n ftype=1
echo "/dev/docker/docker      /var/lib/docker         xfs     defaults        0 0" >> /etc/fstab
mount -a
```

## 虚拟机添加磁盘

```sh
  # 给虚拟机添加磁盘
    qemu-img  create -f qcow2 c2-lain3-etcd.qcow2 100G
    # 挂载qcow2磁盘，如果不指定"--subdriver qcow2",默认为raw格式
    virsh attach-disk c2-lain3  /data2/c2-lain3-etcd.qcow2 vdc --driver qemu --subdriver qcow2 --targetbus virtio --persistent
    # 卸载磁盘
    virsh detach-disk  c2-lain3 vdc --persistent
```

## 删除swap分区，需要相应地修改grub，删除启动项中swap

```sh
swapoff -a
lvremove -Ay /dev/centos/swap 
lvextend +100%FREE centos/root
lvextend -l +100%FREE centos/root
vi /etc/fstab 
vi /etc/default/grub 
grub2-mkconfig > /etc/grub2.cfg 
pvcreate /dev/xvdb 
vgcreate docker /dev/xvdb 
lsblk
vgs
lvcreate docker -l 100%FREE docker
lvs
lvcreate -l 100%FREE docker
lvrename docker/lvol0 docker/docker
lvs
lsblk
mkfs.xfs /dev/docker/docker 
mkdir -p /var/lib/docker
mount /dev/docker/docker /var/lib/docker/
vi /etc/fstab 
lsblk
reboot
```
