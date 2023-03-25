## Домашнее задание "Нагрузочное тестирование и тюнинг PostgreSQL"

## Цель:

-   сделать нагрузочное тестирование PostgreSQL
-   настроить параметры PostgreSQL для достижения максимальной производительности

  
## Описание/Пошаговая инструкция выполнения домашнего задания:

### Развернуть виртуальную машину любым удобным способом  

- Сделано в Yandex Cloud

>_yc compute instance create --name pg-instance --hostname pg-instance --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --metadata-from-file ssh-keys=C:/Users/User/.ssh/sshkeys.txt_

### Поставить на неё PostgreSQL 15 любым способом  
ssh user@84.201.139.251
- Updating your system APT index to refresh the system packages. 

> sudo apt update && sudo apt dist-upgrade -y

> sudo apt install gnupg2 wget vim -y

- Enable PostgreSQL 15 Package Repository

> sudo apt-cache search postgresql | grep postgresql

> sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

> wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null

> sudo apt update -y

- Step 2: Install PostgreSQL 15 Database Server and Client

> sudo apt install postgresql postgresql-client -y

> sudo systemctl enable postgresql

> sudo systemctl start postgresql

> systemctl status postgresql

sudo -u postgres psql -c "SELECT version();"
sudo -u postgres psql

\password

### Настроить кластер PostgreSQL 15 на максимальную производительность не  
обращая внимание на возможные проблемы с надежностью в случае  
аварийной перезагрузки виртуальной машины  


### Нагрузить кластер через утилиту через утилиту pgbench ([https://postgrespro.ru/docs/postgrespro/14/pgbench](https://postgrespro.ru/docs/postgrespro/14/pgbench "https://postgrespro.ru/docs/postgrespro/14/pgbench"))  


### Написать какого значения tps удалось достичь, показать какие параметры в  
какие значения устанавливали и почему  
Задание со *: аналогично протестировать через утилиту  [https://github.com/Percona-Lab/sysbench-tpcc](https://github.com/Percona-Lab/sysbench-tpcc "https://github.com/Percona-Lab/sysbench-tpcc")  (требует установки  
[https://github.com/akopytov/sysbench](https://github.com/akopytov/sysbench "https://github.com/akopytov/sysbench"))



<!--stackedit_data:
eyJoaXN0b3J5IjpbNDQ5Mjc0ODIsLTE0OTQxMzAxNzcsLTEwMD
g4MTUyNjVdfQ==
-->