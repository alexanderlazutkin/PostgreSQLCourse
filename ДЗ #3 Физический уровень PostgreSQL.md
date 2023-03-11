
##  Домашнее задание 3
###  Физический уровень PostgreSQL
>Описание/Пошаговая инструкция выполнения домашнего задания:

##### создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE/ЯО
> Сделано в Yandex Cloud :
> _yc compute instance create --name pg-instance --hostname pg-instance --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --metadata-from-file ssh-keys=C:/Users/User/.ssh/sshkeys.txt_
> ssh user@130.193.41.225
#####  поставьте на нее PostgreSQL 14 через sudo apt
> ssh user@158.160.27.62
#####  проверьте что кластер запущен через sudo -u postgres pg_lsclusters
-   зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым  
    postgres=# create table test(c1 text);  
    postgres=# insert into test values('1');  
    \q
-   остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop
-   создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB
-   добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
-   проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb -  [https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux "https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux")
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
eyJoaXN0b3J5IjpbLTk4NTM0MDEwLDE1OTQ0NzgyODldfQ==
-->