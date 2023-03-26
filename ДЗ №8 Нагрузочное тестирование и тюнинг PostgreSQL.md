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

Написать какого значения tps удалось достичь, показать какие параметры в  
какие значения устанавливали и почему  
Задание со *: аналогично протестировать через утилиту  [https://github.com/Percona-Lab/sysbench-tpcc](https://github.com/Percona-Lab/sysbench-tpcc "https://github.com/Percona-Lab/sysbench-tpcc")  (требует установки  
[https://github.com/akopytov/sysbench](https://github.com/akopytov/sysbench "https://github.com/akopytov/sysbench"))

Поменяем настройки:

 - List ishared_buffers - объем выделенной памяти для PostgreSQL для кеширования. Рекомендовано устанавливать значение в 25% от общей RAM Оставим как было в для pgtune (512MB)

Чтобы отключить передачу изменений через WAL. Это не только поможет сэкономить время архивации и передачи WAL, но и непосредственно ускорит некоторые команды, потому что они не записывают в WAL ничего, если в wal_level установлен уровень minimal и текущая подтранзакция (или транзакция верхнего уровня) создала и опустошила таблицу или индекс, куда затем вносятся изменения.
wal_level = 'minimal', max_wal_senders = 0 - минимизируем объем передаваемый в WAL информации  (необходимую только для восстановления после сбоя или аварийного отключения)
archive_mode = Off. Полные сегменты WAL передаются в хранилище архива при ON, ALWAYS.

max_wal_size - Максимальный размер, до которого может вырастать WAL во время автоматических контрольных точек. Увеличим до 4GB
min_wal_size - Пока WAL занимает на диске меньше этого объёма, старые файлы WAL в контрольных точках всегда перерабатываются, а не удаляются. Увеличим до 16GB
maintenance_work_mem - максимальный объём памяти для операций обслуживания БД. Установка большого значения помогает в таких задачах, как VACUUM, RESTORE, CREATE INDEX, ADD FOREIGN KEY и ALTER TABLE. Увеличим до 256MB.
effective_io_concurrency - задаёт допустимое число параллельных операций ввода/вывода одновременно. Оставим как было в для pgtune (2)
effective_cache_size - сообщает оптимизатору объем кеша, доступный в ядре. Если значение этого параметра установлено слишком низким, планировщик запросов может принять решение не использовать некоторые индексы, даже если они будут полезны. Поэтому установка большого значения всегда имеет смысл. Оставим как было в для pgtune  (1536MB)

Увеличим время ожидания контрольной точки для меньшей нагрузки.
checkpoint_timeout - Максимальное время между автоматическими контрольными точками в WAL. Увеличим до 1часа
checkpoint_completion_target = Задаёт целевое время для завершения процедуры контрольной точки, как коэффициент для общего времени между контрольными точками. Установим 0.9

autovacuum - отключаем автовакуум только в нашем случае для ускорения транзакций. Рекомендовано не отключать его на прод-среде.
random_page_cost - определяет приблизительную стоимость чтения одной произвольной страницы с диска. Оставим как было в для pgtune  (4)

Группа показателей для получения максимального показателя tps. Отключим их все (Off)

synchronous_commit - в значении ON определяет, что фиксация транзакции будет ожидать записи WAL. 
fsync - в состоянии ON изменения будут записаны на диск физически, выполняя системные вызовы fsync() или другими подобными методами (см. wal_sync_method). Это даёт гарантию, что кластер баз данных сможет вернуться в согласованное состояние после сбоя оборудования или операционной системы. 
full_page_writes - записывает в журнал полный образ страницы при первом ее изменении после начала контрольной точки

При установке настроек производительность увеличилась на 3063 - 638 = 2425 tps или в 3.8 раза.

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM0MjE0MjkwMywzNTM2MzIwMzAsLTE1ND
g0NzU1NTYsLTU3Njg3OTM2OSwxNzgxOTUyMjYyLC0xNDk0MTMw
MTc3LC0xMDA4ODE1MjY1XX0=
-->