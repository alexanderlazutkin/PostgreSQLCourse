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

Сделаем первоначальное тестирование с дефолтными настройками :

sudo -u postgres pgbench -i testdb
sudo -u postgres pgbench -c8 -P 10 -T 120 -U postgres testdb

У нас получилось, что для стандартной настройки при помощи pgbench tps составляет 617.

Изменим параметры кластера  /etc/postgresql/15/main/postgresql.conf при помощи утилиты pgtune (https://pgtune.leopard.in.ua/):

sudo -u postgres nano /etc/postgresql/15/main/postgresql.conf

#DB Version: 15
#OS Type: linux
#DB Type: oltp
#Total Memory (RAM): 2 GB
#CPUs num: 2
#Data Storage: hdd


shared_buffers = 512MB
effective_cache_size = 1536MB
maintenance_work_mem = 128MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 2621kB
min_wal_size = 2GB
max_wal_size = 8GB

--sudo systemctl restart postgresql

sudo pg_ctlcluster 15 main reload
sudo -u postgres 

Выполним еще раз pgbench и получаем следующие данные:
sudo -u postgres pgbench -c8 -P 10 -T 120 -U postgres testdb

При установке рекомендованных настроек мы выиграли в производительности всего 638 -617 = 11 tps.


### Настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае  аварийной перезагрузки виртуальной машины  

Изменим параметры кластера  /etc/postgresql/15/main/postgresql.conf при помощи утилиты pgtune (https://pgtune.leopard.in.ua/):

### Написать какого значения tps удалось достичь, показать какие параметры в  
какие значения устанавливали и почему  
Задание со *: аналогично протестировать через утилиту  [https://github.com/Percona-Lab/sysbench-tpcc](https://github.com/Percona-Lab/sysbench-tpcc "https://github.com/Percona-Lab/sysbench-tpcc")  (требует установки  
[https://github.com/akopytov/sysbench](https://github.com/akopytov/sysbench "https://github.com/akopytov/sysbench"))



<!--stackedit_data:
eyJoaXN0b3J5IjpbMTAzMjg2NTM3MywzNTM2MzIwMzAsLTE1ND
g0NzU1NTYsLTU3Njg3OTM2OSwxNzgxOTUyMjYyLC0xNDk0MTMw
MTc3LC0xMDA4ODE1MjY1XX0=
-->