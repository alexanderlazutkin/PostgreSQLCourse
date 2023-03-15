
##  Домашнее задание 3
###  Физический уровень PostgreSQL
>Описание/Пошаговая инструкция выполнения домашнего задания:

##### создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE/ЯО
> Сделано в Yandex Cloud :
> _yc compute instance create --name pg-instance --hostname pg-instance --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --metadata-from-file ssh-keys=C:/Users/User/.ssh/sshkeys.txt_
> ssh user@158.160.11.34
#####  поставьте на нее PostgreSQL 14 через sudo apt https://techviewleo.com/how-to-install-postgresql-database-on-ubuntu/
sudo apt -y install gnupg2 wget vim
sudo apt -y update
sudo apt -y install postgresql-14

user@postgresql:~$ systemctl status postgresql

>postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2023-03-11 10:45:06 UTC; 33s ago
    Process: 42418 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 42418 (code=exited, status=0/SUCCESS)
   
sudo -u postgres psql -c "SELECT version();"  

>PostgreSQL 14.7 (Ubuntu 14.7-0ubuntu0.22.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0, 64-bit
> 
#####  проверьте что кластер запущен через sudo -u postgres pg_lsclusters

user@postgresql:~$ sudo -u postgres pg_lsclusters

>Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log

##### зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым  

sudo -u postgres psql

postgres=# create table test(c1 text);

postgres=# insert into test values('1');

postgres-# \q

##### остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop

user@postgresql:~$ sudo -u postgres pg_ctlcluster 14 main stop

>Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:

#####   создайте новый standard persistent диск GKE через ЯО
- остановлена ВМ, добавлен диск и присоединенен, ВМ запущена

user@postgres:~$ sudo lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL

>NAME   FSTYPE     SIZE MOUNTPOINT        LABEL
loop0  squashfs  63.3M /snap/core20/1828
loop1  squashfs  61.9M /snap/core20/1405
loop2  squashfs 111.9M /snap/lxd/24322
loop3  squashfs  79.9M /snap/lxd/22923
loop4  squashfs  49.8M /snap/snapd/18357
vda                15G
├─vda1              1M
└─vda2 ext4        15G /
vdb                20G

#####  проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb -  [https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux "https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux")
https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
Подготовка к разметке нового диска
sudo apt update
sudo apt install parted

user@postgresql:~$ sudo parted -l | grep Error
Error: /dev/vdb: unrecognised disk label

Разметка нового диска для GPT
sudo parted /dev/vdb mklabel gpt

Создание нового раздела
>sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%

user@postgresql:~$ lsblk
>NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0  63.3M  1 loop /snap/core20/1828
loop1    7:1    0  61.9M  1 loop /snap/core20/1405
loop2    7:2    0 111.9M  1 loop /snap/lxd/24322
loop3    7:3    0  79.9M  1 loop /snap/lxd/22923
loop4    7:4    0  49.8M  1 loop /snap/snapd/18357
vda    252:0    0    15G  0 disk
├─vda1 252:1    0     1M  0 part
└─vda2 252:2    0    15G  0 part /
vdb    252:16   0    20G  0 disk
└─vdb1 252:17   0    20G  0 part

Create a Filesystem on the New Partition
>sudo mkfs.ext4 -L datapartition /dev/vdb

user@postgresql:~$ sudo lsblk --fs
>NAME   FSTYPE   FSVER LABEL         UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0  squashfs 4.0                                                            0   100% /snap/core20/1828
loop1  squashfs 4.0                                                            0   100% /snap/core20/1405
loop2  squashfs 4.0                                                            0   100% /snap/lxd/22923
loop3  squashfs 4.0                                                            0   100% /snap/lxd/24322
loop4  squashfs 4.0                                                            0   100% /snap/snapd/18357
vda
├─vda1
└─vda2 ext4     1.0                 82aeea96-6d42-49e6-85d5-9071d3c9b6aa    9.8G    29% /
vdb    ext4     1.0   datapartition ecceaf63-6127-4524-a8ab-b9b38db37845

user@postgresql:~$ sudo lsblk -o NAME,FSTYPE,LABEL,UUID,MOUNTPOINT
>NAME   FSTYPE   LABEL         UUID                                 MOUNTPOINT
loop0  squashfs                                                    /snap/core20/1828
loop1  squashfs                                                    /snap/core20/1405
loop2  squashfs                                                    /snap/lxd/24322
loop3  squashfs                                                    /snap/lxd/22923
loop4  squashfs                                                    /snap/snapd/18357
vda
├─vda1
└─vda2 ext4                   82aeea96-6d42-49e6-85d5-9071d3c9b6aa /
vdb    ext4     datapartition ecceaf63-6127-4524-a8ab-b9b38db37845

Mount the New Filesystem
>sudo mkdir -p /mnt/data
sudo mount -o defaults /dev/vdb /mnt/data
sudo mount -a


user@postgresql:~$ df -h -x tmpfs
>Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        15G  4.2G  9.9G  30% /
/dev/vdb         20G   24K   19G   1% /mnt/data

sudo vi /etc/fstab
добавить /dev/vdb /mnt/data ext4 defaults 0 1

##### перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)
user@postgresql:~$ sudo reboot now
>Connection to 51.250.29.213 closed by remote host.
Connection to 51.250.29.213 closed.

user@postgresql:~$ df -h -x tmpfs
>Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        15G  4.2G  9.9G  30% /
/dev/vdb         20G   24K   19G   1% /mnt/data

##### сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
sudo chown -R postgres:postgres /mnt/data/

##### перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/14 /mnt/data
sudo chown -R postgres:postgres /mnt/data/
sudo systemctl stop postgresql
sudo systemctl status postgresql
sudo -u postgres mv /var/lib/postgresql/14 /mnt/data

--sudo rsync -av /var/lib/postgresql /mnt/data
--sudo mv /var/lib/postgresql/14/main /var/lib/postgresql/14/main.bak

##### попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
#####  напишите получилось или нет и почему
user@postgresql:~$ sudo -u postgres pg_ctlcluster 14 main start
>Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@14-main
Error: Could not open logfile /var/log/postgresql/postgresql-14-main.log
Error: /usr/lib/postgresql/14/bin/pg_ctl /usr/lib/postgresql/14/bin/pg_ctl start -D /var/lib/postgresql/14/main -l /var/log/postgresql/postgresql-14-main.log -s -o  -c config_file="/etc/postgresql/14/main/postgresql.conf"  exited with status 1:
Ошибка, т.к. предыдущий путь не изменен в конфигурации. Нужно поправить

##### задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/14/main который надо поменять и поменяйте его

sudo nano /etc/postgresql/14/main/postgresql.conf

##### напишите что и почему поменяли
data_directory = '/mnt/data/var/lib/postgresql/14/main'
>указал новый путь, куда переместили файл

##### попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start напишите получилось или нет и почему
sudo systemctl start postgresql
sudo systemctl status postgresql

##### зайдите через через psql и проверьте содержимое ранее созданной таблицы
созданная таблица на месте

postgres=# select * from test;
 c1
 ----
 1
(1 row)
--sudo rm -Rf /var/lib/postgresql/10/main.bak

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg4NjI0OTM4NSwtNjY1MTU5ODQ3LDE1OT
g1ODIwNDIsLTE1NjI0MzA5NDksMTc0NjEyNTMxMiwtNTQ1OTY0
OTEwLC0xMzUxMTA1MTkwLC04Mjk4NTQ2NSwyMTIwNTI3Njk4LC
05ODEyMDQwNTcsNTE2MDk5MjYyLDE2NzY3NzU2NTMsLTE4MDI0
NTA3MTEsLTEwMzU3NDYwNDMsMTkwMTE5Mzg5OCwtMTU3ODYyMD
U3OCwxNTk0NDc4Mjg5XX0=
-->