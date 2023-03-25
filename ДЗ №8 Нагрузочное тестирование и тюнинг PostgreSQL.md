## Домашнее задание "Нагрузочное тестирование и тюнинг PostgreSQL"

## Цель:

-   сделать нагрузочное тестирование PostgreSQL
-   настроить параметры PostgreSQL для достижения максимальной производительности

  
## Описание/Пошаговая инструкция выполнения домашнего задания:

### Развернуть виртуальную машину любым удобным способом  

- Сделано в Yandex Cloud

>_yc compute instance create --name pg-instance --hostname pg-instance --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --metadata-from-file ssh-keys=C:/Users/User/.ssh/sshkeys.txt_

### Поставить на неё PostgreSQL 15 любым способом  

ssh user@51.250.96.46
sudo apt update
sudo apt --list upgradable
sudo apt upgrade
sudo apt install dirmngr ca-certificates software-properties-common gnupg gnupg2 apt-transport-https curl -y

curl -fSsL https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | sudo tee /usr/share/keyrings/postgresql.gpg > /dev/null

echo deb [arch=amd64,arm64,ppc64el signed-by=/usr/share/keyrings/postgresql.gpg] http://apt.postgresql.org/pub/repos/apt/ jammy-pgdg main | sudo tee -a /etc/apt/sources.list.d/postgresql.list



Step 2: Install PostgreSQL 15 Database Server and Client
sudo apt-get update
sudo apt install postgresql-client-15 postgresql-15 -y
sudo apt install postgresql-client postgresql -y

sudo systemctl enable postgresql
sudo systemctl start postgresql
systemctl status postgresql

sudo -u postgres psql -c "SELECT version();"
sudo -u postgres psql

\password

Создадим отдельную базу данных и роль для тестирования нагрузки:
create database testdb;
\c testdb

### Настроить кластер PostgreSQL 15 на максимальную производительность не  
обращая внимание на возможные проблемы с надежностью в случае  
аварийной перезагрузки виртуальной машины  


### Нагрузить кластер через утилиту через утилиту pgbench ([https://postgrespro.ru/docs/postgrespro/14/pgbench](https://postgrespro.ru/docs/postgrespro/14/pgbench "https://postgrespro.ru/docs/postgrespro/14/pgbench"))  


### Написать какого значения tps удалось достичь, показать какие параметры в  
какие значения устанавливали и почему  
Задание со *: аналогично протестировать через утилиту  [https://github.com/Percona-Lab/sysbench-tpcc](https://github.com/Percona-Lab/sysbench-tpcc "https://github.com/Percona-Lab/sysbench-tpcc")  (требует установки  
[https://github.com/akopytov/sysbench](https://github.com/akopytov/sysbench "https://github.com/akopytov/sysbench"))



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NDg0NzU1NTYsLTU3Njg3OTM2OSwxNz
gxOTUyMjYyLC0xNDk0MTMwMTc3LC0xMDA4ODE1MjY1XX0=
-->