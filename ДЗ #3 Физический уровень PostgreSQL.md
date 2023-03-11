
##  Домашнее задание 3
###  Физический уровень PostgreSQL
>Описание/Пошаговая инструкция выполнения домашнего задания:

##### создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE/ЯО
> Сделано в Yandex Cloud :
> _yc compute instance create --name pg-instance --hostname pg-instance --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --metadata-from-file ssh-keys=C:/Users/User/.ssh/sshkeys.txt_
> ssh user@130.193.41.225
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

##### остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop
postgres-# \q
>user@postgresql:~$ sudo -u postgres pg_ctlcluster 14 main stop
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:

#####   создайте новый standard persistent диск GKE через ЯО
- остановлена ВМ, добавлен диск и присоединенен, ВМ запущена
> user@postgresql:~$ sudo lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL
NAME   FSTYPE     SIZE MOUNTPOINT        LABEL
loop0  squashfs  61.9M /snap/core20/1405
loop1  squashfs  63.3M /snap/core20/1828
loop2  squashfs  79.9M /snap/lxd/22923
loop3  squashfs 111.9M /snap/lxd/24322
loop4  squashfs  49.8M /snap/snapd/18357
vda                15G
├─vda1              1M
└─vda2 ext4        15G /
vdb                20G
#####  проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb -  [https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux "https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux")

-   перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)
-   сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
-   перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/14 /mnt/data
-   попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
-   напишите получилось или нет и почему
-   задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/14/main который надо поменять и поменяйте его
-   напишите что и почему поменяли
-   попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
-   напишите получилось или нет и почему
-   зайдите через через psql и проверьте содержимое ранее созданной таблицы
-   задание со звездочкой *: не удаляя существующий инстанс ВМ сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY3Njc3NTY1MywtMTgwMjQ1MDcxMSwtMT
AzNTc0NjA0MywxOTAxMTkzODk4LC0xNTc4NjIwNTc4LDE1OTQ0
NzgyODldfQ==
-->