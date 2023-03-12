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
user@postgres:~$ sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres
pgbench (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
starting vacuum...end.
progress: 60.0 s, 580.2 tps, lat 13.782 ms stddev 9.852
progress: 120.0 s, 587.4 tps, lat 13.614 ms stddev 9.975
progress: 180.0 s, 578.0 tps, lat 13.845 ms stddev 10.054
progress: 240.0 s, 582.5 tps, lat 13.733 ms stddev 9.995
progress: 300.0 s, 596.4 tps, lat 13.415 ms stddev 9.731
progress: 360.0 s, 533.3 tps, lat 15.000 ms stddev 11.676
progress: 420.0 s, 572.5 tps, lat 13.974 ms stddev 9.837
progress: 480.0 s, 532.8 tps, lat 15.015 ms stddev 10.778
progress: 540.0 s, 579.9 tps, lat 13.796 ms stddev 9.632
progress: 600.0 s, 566.3 tps, lat 14.124 ms stddev 9.815
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 342560
latency average = 14.012 ms
latency stddev = 10.146 ms
initial connection time = 16.702 ms
tps = 570.925569 (without initial connection time)

#### дальше настроить autovacuum максимально эффективно
-   построить график по получившимся значениям так чтобы получить максимально ровное значение tps
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyMTc3ODI3MzQsMTMwNTc0ODU0LC05OT
A5OTkyOSwxMTY0NzM0NTM0XX0=
-->