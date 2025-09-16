# NFS File Sharing on Ubuntu Server with LVM Storage Management
## **Introduction**

This Project Describe Deployment of Network File System **(NFS)** Server on Ubuntu Server with Storage Mangament by Logical Volum Manager **(LVM)**.

## **Prerequisites**

Ubuntu Server 24.04.
Root or sudo Access.
Network Connection with Server and Clients.
nfs-kernel-server and nfs-common and lvm2 packages.

## **LVM Configuration**

I have two 25GB Disk on my Server and ubuntu define it by "sdb" and "sdc":

```bash
root@nfs-srv:~# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                    8:0    0  100G  0 disk 
├─sda1                 8:1    0    1M  0 part 
├─sda2                 8:2    0    2G  0 part /boot
└─sda3                 8:3    0   98G  0 part 
  └─ubuntu--vg-lv--0 252:0    0   98G  0 lvm  /
sdb                    8:16   0   25G  0 disk 
sdc                    8:32   0   25G  0 disk 
sr0                   11:0    1 1024M  0 rom  
```
First we need to create a physical volum **(PV)**:
```bash
pvcreate /dev/sbd /dev/sdc
```
and then create a volum group **(VG)**:
```bash
vgcreate nfs-vg /dev/sdb /dev/sdc
```
after this create a Logical volume **(LV)**:
```bash
lvcreate -n nfs-lv -l 100%FREE nfs-vg
```
set **ext4** filesystem for this volume:
```bash
mkfs.ext4 /dev/nfs-vg/nfs-lv
```
make directory  **/opt/nfs** to mount this volume on this 
folder.
because connection between Server and Client is trust, we set open permission on this folder
```bash
chown nobody:nogroup /opt/nfs/
chmod 777 /opt/nfs/
```

finde UUID of this disk for permanent mount on server:
```bash
blkid /dev/nfs-vg/nfs-lv
```
edit **/etc/fstab** and add this line end of file:
```bash
UUID=<blkid> /opt/nfs ext4 defaults 0 0
```
and then enter this command to apply this
```bash
mount -a
```
and now lvm set succesfully :)


## **NFS Server Installation**
install packages:
```bash
apt install nfs-kernel-server nfs-common -y
```

edit **/etc/exports** for set config add this line end of file:
```bash
/opt/nfs <network subnet between server and clients>(rw,sync,no_subtree_check)
```
then apply config and restart service:
```bash
exportfs -r
exportfs -a
systemctl restart nfs-kernel-server.service
```
now NFS Server configured completely

## **NFS Client Configuration**
install packages:
```bash
apt install nfs-common -y
```

make directory  **/opt/nfs** to mount this volume on this, Similar Server
folder.
because connection between Server and Client is trust, we set open permission on this folder
```bash
chown nobody:nogroup /opt/nfs/
chmod 777 /opt/nfs/
```
for permanent mount nfs server to this client, edit **/etc/fstab** and add this line end of file:
```bash
<NFS Server IP>:/opt/nfs /opt/nfs  nfs defaults 0 0
```
and then enter this command to apply this
```bash
mount -a
```
and now nfs client mount to nfs server!

## **Testing & Verification**
