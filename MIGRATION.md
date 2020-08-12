Migration
=========


MySQL
-----

```
ALTER TABLE user CHANGE geburtstag geburtstag DATE NOT NULL DEFAULT '1970-01-01';
```

POSTFIX
-------

prepare chroot

```
cp -f /etc/services /data/spool/postfix/etc/
cp -f /etc/resolv.conf /data/spool/postfix/etc/
```

Data
----

Backup the old /data partition
/etc/fstab:

```
//gnuubackup.cifs.hidrive.strato.com/root /hidrive cifs credentials=/etc/samba/hidrive.credentials 0  0
```

/etc/samba/hidrive.credentials
```
username=xxxxxx
password=xxxxxx
```

```
systemctl stop postfix
systemctl stop innd
systemctl stop apache2
systemctl stop xinetd
mysql gnuu
> show grants for 'gnuuweb'@'localhost';
# remark permissions
>quit
mysqldump gnuu > /data/mysqlbackup/gnuu.sql
systemctl stop mysql
mount /hidrive
cd /hidrive/users/xxxx
tar cvfz data-20200804.tgz /data
```

takes ~45 min

another option:

```
rsync -rltDvze "ssh -i /root/.ssh/id_rsa" "backup-`date +%F`.tgz" xxxxx@rsync.hidrive.strato.com:/users/xxxx/
```


openebs-standalone PVC are accessable under normal device path:

```
sda      8:0    0    8G  0 disk /var/lib/kubelet/pods/f235fe03-3953-40c6-ae19-87635bb89fa7/volumes/kubernetes.io~iscsi/pvc-3f9
sdb      8:16   0    1G  0 disk /var/lib/kubelet/pods/3d5c111f-ccfd-4c0a-8f31-62721301f1e1/volumes/kubernetes.io~iscsi/pvc-eaf
sdc      8:32   0   10G  0 disk /var/lib/kubelet/pods/d0658858-3d69-46d6-a20c-e265ce0c78f4/volumes/kubernetes.io~iscsi/pvc-094
sdd      8:48   0    8G  0 disk /var/lib/kubelet/pods/31ed1be3-c728-410e-81a9-42c81f5eb2e1/volumes/kubernetes.io~iscsi/pvc-614
```
The device can be mount temporary (!) with mount /dev/sda /mnt (in this time the POD has no access to the volume)
copy data back 

```
mount /dev/sda /mnt
cd /mnt
tar xvfz /hidrive/users/xxxx/data-20200804.tgz 
```

MySQL Restore from backup
-------------------------

```
kubectl exec -it mysql-5cbf8684cc-jfwh2 -- mysql -e "create database gnuu"
kubectl exec -it mysql-5cbf8684cc-jfwh2 -- mysql gnuu < /data/mysqlbackup/gnuu.sql
# create app user
kubectl exec -it mysql-5cbf8684cc-jfwh2 -- mysql -e "flush privileges"
```
