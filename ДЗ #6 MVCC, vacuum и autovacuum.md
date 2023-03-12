## Домашнее задание

Настройка autovacuum с учетом оптимальной производительности

### Цель:
-   запустить нагрузочный тест pgbench
-   настроить параметры autovacuum для достижения максимального уровня устойчивой производительности

  

### Описание/Пошаговая инструкция выполнения домашнего задания:

####  создать GCE инстанс типа e2-medium и диском 10GB
####   установить на него PostgreSQL 14 с дефолтными настройками
ssh user@158.160.17.236
sudo apt -y install gnupg2 wget vim
sudo apt -y update
sudo apt -y install postgresql-14

sudo -u postgres psql
ALTER USER postgres WITH PASSWORD 'postgres';
postgres-# \c postgres

#### Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла

SELECT name, setting, unit FROM pg_settings WHERE context = 'postmaster';
ALTER SYSTEM SET max_connections TO 40;
ALTER SYSTEM SET shared_buffers TO '1GB';
ALTER SYSTEM SET effective_cache_size TO '3GB';
ALTER SYSTEM SET maintenance_work_mem TO '512MB';
ALTER SYSTEM SET checkpoint_completion_target TO 0.9;
ALTER SYSTEM SET wal_buffers TO '16MB';
ALTER SYSTEM SET default_statistics_target TO 500;
ALTER SYSTEM SET random_page_cost TO 4;
ALTER SYSTEM SET effective_io_concurrency TO 2;
ALTER SYSTEM SET work_mem TO '6553kB';
ALTER SYSTEM SET min_wal_size TO '4GB';
ALTER SYSTEM SET max_wal_size TO '16GB';
\q
sudo systemctl restart postgresql

#### выполнить pgbench -i postgres
user@postgres:~$ sudo -u postgres pgbench -i postgres
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.07 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.52 s (drop tables 0.00 s, create tables 0.03 s, client-side generate 0.35 s, vacuum 0.06 s, primary keys 0.09 s).

#### запустить pgbench -c8 -P 60 -T 600 -U postgres postgres и   дать отработать до конца

-   дальше настроить autovacuum максимально эффективно
-   построить график по получившимся значениям
-   так чтобы получить максимально ровное значение tps
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMwNTc0ODU0LC05OTA5OTkyOSwxMTY0Nz
M0NTM0XX0=
-->