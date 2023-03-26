## Домашнее задание "Нагрузочное тестирование и тюнинг PostgreSQL"

## Цель:

-   сделать нагрузочное тестирование PostgreSQL
-   настроить параметры PostgreSQL для достижения максимальной производительности

  
## Описание/Пошаговая инструкция выполнения домашнего задания:

### Развернуть виртуальную машину любым удобным способом  

- Сделано в Yandex Cloud

>_yc compute instance create --name pg-instance --hostname pg-instance --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --metadata-from-file ssh-keys=C:/Users/User/.ssh/sshkeys.txt_

### Поставить на неё PostgreSQL 15 любым способом  


ssh user@178.154.199.252
sudo apt update
sudo apt --list upgradable
sudo apt upgrade
sudo apt install dirmngr ca-certificates software-properties-common gnupg gnupg2 apt-transport-https curl -y

curl -fSsL https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | sudo tee /usr/share/keyrings/postgresql.gpg > /dev/null

echo deb [arch=amd64,arm64,ppc64el signed-by=/usr/share/keyrings/postgresql.gpg] http://apt.postgresql.org/pub/repos/apt/ jammy-pgdg main | sudo tee -a /etc/apt/sources.list.d/postgresql.list



Step 2: Install PostgreSQL 15 Database Server and Client
sudo apt-get update
sudo apt install postgresql-client-15 postgresql-15 -y

sudo systemctl enable postgresql
sudo systemctl start postgresql
systemctl status postgresql

sudo -u postgres psql -c "SELECT version();"
> PostgreSQL 15.2 (Ubuntu 15.2-1.pgdg22.04+1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0, 64-bit

sudo -u postgres psql

\password


create database testdb;
\c testdb



### Нагрузить кластер через утилиту через утилиту pgbench ([https://postgrespro.ru/docs/postgrespro/14/pgbench](https://postgrespro.ru/docs/postgrespro/14/pgbench "https://postgrespro.ru/docs/postgrespro/14/pgbench"))  


### Настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае  аварийной перезагрузки виртуальной машины  

Сделаем первоначальное тестирование с дефолтными настройками :

sudo -u postgres pgbench -i testdb
sudo -u postgres pgbench -c8 -P 10 -T 120 -U postgres testdb

У нас получилось, что для стандартной настройки при помощи pgbench tps составляет 617.

### Написать какого значения tps удалось достичь, показать какие параметры в  
какие значения устанавливали и почему  
Задание со *: аналогично протестировать через утилиту  [https://github.com/Percona-Lab/sysbench-tpcc](https://github.com/Percona-Lab/sysbench-tpcc "https://github.com/Percona-Lab/sysbench-tpcc")  (требует установки  
[https://github.com/akopytov/sysbench](https://github.com/akopytov/sysbench "https://github.com/akopytov/sysbench"))



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2OTU0NzQ1MTMsMzUzNjMyMDMwLC0xNT
Q4NDc1NTU2LC01NzY4NzkzNjksMTc4MTk1MjI2MiwtMTQ5NDEz
MDE3NywtMTAwODgxNTI2NV19
-->