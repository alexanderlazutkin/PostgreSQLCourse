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

>creating tables...
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

>transaction type: <builtin: TPC-B (sort of)>
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

#### дальше настроить autovacuum максимально эффективно. построить график по получившимся значениям так чтобы получить максимально ровное значение tps

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

>transaction type: <builtin: TPC-B (sort of)>
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

sudo -u postgres psql

SELECT name, setting, context, short_desc FROM pg_settings WHERE name like 'autovacuum%';

SELECT name, setting, context, short_desc FROM pg_settings WHERE name like 'vacuum%';

ALTER SYSTEM SET autovacuum_naptime TO 2; -- чтобы долго не ждать

ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.05;  

ALTER SYSTEM SET autovacuum_vacuum_threshold = 0;

SELECT pg_reload_conf();



user@postgres:~$ sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres

pgbench (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))

starting vacuum...end.

progress: 60.0 s, 565.8 tps, lat 14.134 ms stddev 10.636

progress: 120.0 s, 579.1 tps, lat 13.813 ms stddev 11.009

progress: 180.0 s, 574.2 tps, lat 13.930 ms stddev 11.443

progress: 240.0 s, 555.6 tps, lat 14.398 ms stddev 11.865

progress: 300.0 s, 591.7 tps, lat 13.520 ms stddev 10.071

progress: 360.0 s, 574.7 tps, lat 13.922 ms stddev 10.374

progress: 420.0 s, 603.1 tps, lat 13.265 ms stddev 10.479

progress: 480.0 s, 565.3 tps, lat 14.151 ms stddev 11.172

progress: 540.0 s, 599.4 tps, lat 13.345 ms stddev 9.563

progress: 600.0 s, 581.1 tps, lat 13.765 ms stddev 10.388

>transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 347412
latency average = 13.816 ms
latency stddev = 10.713 ms
initial connection time = 17.779 ms
tps = 579.015236 (without initial connection time)


sudo -u postgres psql

ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.03;  

ALTER SYSTEM SET autovacuum_analyze_scale_factor = 0.02;

ALTER SYSTEM SET autovacuum_analyze_threshold = 0;

SELECT pg_reload_conf();

\q



user@postgres:~$ sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres

pgbench (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))

starting vacuum...end.

progress: 60.0 s, 575.9 tps, lat 13.885 ms stddev 10.802

progress: 120.0 s, 596.9 tps, lat 13.403 ms stddev 10.206

progress: 180.0 s, 586.3 tps, lat 13.645 ms stddev 10.951

progress: 240.0 s, 586.2 tps, lat 13.646 ms stddev 10.786

progress: 300.0 s, 599.7 tps, lat 13.340 ms stddev 9.882

progress: 360.0 s, 576.7 tps, lat 13.872 ms stddev 10.579

progress: 420.0 s, 589.0 tps, lat 13.581 ms stddev 10.142

progress: 480.0 s, 562.8 tps, lat 14.214 ms stddev 11.665

progress: 540.0 s, 596.8 tps, lat 13.405 ms stddev 9.785

progress: 600.0 s, 582.6 tps, lat 13.730 ms stddev 10.093

>transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 351180
latency average = 13.668 ms
latency stddev = 10.498 ms
initial connection time = 17.589 ms
tps = 585.292981 (without initial connection time)
user@postgres:~$

ssh user@158.160.17.236

sudo -u postgres psql

ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.04;

SELECT pg_reload_conf();

\q
user@postgres:~$ sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres

pgbench (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))

starting vacuum...end.

progress: 60.0 s, 580.0 tps, lat 13.783 ms stddev 9.938

progress: 120.0 s, 566.9 tps, lat 14.114 ms stddev 10.032

progress: 180.0 s, 595.7 tps, lat 13.428 ms stddev 9.196

progress: 240.0 s, 589.9 tps, lat 13.564 ms stddev 9.470

progress: 300.0 s, 553.7 tps, lat 14.447 ms stddev 10.133

progress: 360.0 s, 548.0 tps, lat 14.597 ms stddev 10.610

progress: 420.0 s, 555.9 tps, lat 14.392 ms stddev 10.076

progress: 480.0 s, 573.3 tps, lat 13.953 ms stddev 9.619

progress: 540.0 s, 558.1 tps, lat 14.334 ms stddev 9.918

progress: 600.0 s, 542.3 tps, lat 14.751 ms stddev 10.488

>transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 339839
latency average = 14.124 ms
latency stddev = 9.954 ms
initial connection time = 17.758 ms
tps = 566.390073 (without initial connection time)


### Измененные параметры в сравнении с установленными по умолчанию
postgres=# select name,setting as current_value,reset_val,boot_val as original_default,sourcefile,sourceline from pg_settings where source <> 'default' and name like '%autovacuum%';

name  | current_value | reset_val | original_default | sourcefile  | sourceline


 autovacuum_analyze_scale_factor | 0.02          | 0.02      | 0.1              | /var/lib/postgresql/14/main/postgresql.auto.conf |         16
 autovacuum_analyze_threshold    | 0             | 0         | 50               | /var/lib/postgresql/14/main/postgresql.auto.conf |         17
 autovacuum_max_workers          | 3             | 3         | 3                | /var/lib/postgresql/14/main/postgresql.auto.conf |         20
 autovacuum_naptime              | 10            | 10        | 60               | /var/lib/postgresql/14/main/postgresql.auto.conf |         19
 autovacuum_vacuum_cost_delay    | 1             | 1         | 2                | /var/lib/postgresql/14/main/postgresql.auto.conf |         18
 autovacuum_vacuum_scale_factor  | 0.04          | 0.04      | 0.2              | /var/lib/postgresql/14/main/postgresql.auto.conf |         21
 autovacuum_vacuum_threshold     | 0             | 0         | 50               | /var/lib/postgresql/14/main/postgresql.auto.conf |         15
 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMjYzMjUzODIsLTIxMTM4MTc3NjQsLT
E0OTM5NTUxNDgsLTE3MDQzNjM2MDgsMTAxNzM4NjMzOSw5NjQ5
NzkyOTUsLTEyODc0NDU2NzYsLTEyMTc3ODI3MzQsMTMwNTc0OD
U0LC05OTA5OTkyOSwxMTY0NzM0NTM0XX0=
-->