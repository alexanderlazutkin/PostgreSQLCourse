# Логический уровень PostgreSQL
(https://otus.ru/learning/210735/#)
Описание/Пошаговая инструкция выполнения домашнего задания:

### 1 создайте новый кластер PostgresSQL 14  
sudo apt -y install gnupg2 wget vim
sudo apt -y update
sudo apt -y install postgresql-14
### 2 зайдите в созданный кластер под пользователем postgres  
sudo -u postgres psql
ALTER USER postgres WITH PASSWORD 'postgres';

### 3 создайте новую базу данных testdb  
psql -h localhost -U postgres
>postgres=# CREATE DATABASE testdb;
CREATE DATABASE

postgres=# \q


### 4 зайдите в созданную базу данных под пользователем postgres  
user@postgrevm:~$ psql -h localhost -d testdb -U postgres
>Password for user postgres:
psql (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.
testdb=#

или так:
testdb=# \c testdb
>SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "testdb" as user "postgres".
testdb=#

### 5 создайте новую схему testnm  
CREATE SCHEMA testnm;
### 6 создайте новую таблицу t1 с одной колонкой c1 типа integer  
CREATE TABLE testnm.t1(c1 integer);
### 7 вставьте строку со значением c1=1  
INSERT INTO testnm.t1 values(1);
### 8 создайте новую роль readonly  
CREATE role readonly;
### 9 дайте новой роли право на подключение к базе данных testdb  
grant connect on DATABASE testdb TO readonly;
### 10 дайте новой роли право на использование схемы testnm  
grant usage on SCHEMA testnm to readonly;
### 11 дайте новой роли право на select для всех таблиц схемы testnm  
grant SELECT on all TABLEs in SCHEMA testnm TO readonly;
### 12 создайте пользователя testread с паролем test123  
CREATE USER testread with password 'test123';
### 13 дайте роль readonly пользователю testread  
grant readonly TO testread;
### 14 зайдите под пользователем testread в базу данных testdb  
\c testdb testread
### 15 сделайте select * from t1;  
получилось
testdb=# SELECT * FROM testnm.t1;
с1
----
  1
(1 row)
### 16 получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)  
#### в названии таблицы пропустили схему. в шпаркалке вы сознательно повели по ложному следу и таблицв создалась в схеме по умолчанию PUBLIC. Ай-ай-ай
>
17 напишите что именно произошло в тексте домашнего задания  
18 у вас есть идеи почему? ведь права то дали?  
19 посмотрите на список таблиц  
20 подсказка в шпаргалке под пунктом 20  
21 а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)  

22 вернитесь в базу данных testdb под пользователем postgres  
23 удалите таблицу t1  
24 создайте ее заново но уже с явным указанием имени схемы testnm  
25 вставьте строку со значением c1=1  
26 зайдите под пользователем testread в базу данных testdb  
27 сделайте select * from testnm.t1;  
### 28 получилось?  
>testdb=# \c testdb testread;
Password for user testread:
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "testdb" as user "testread".
testdb=> select * from testnm.t1;
ERROR:  permission denied for table t1

29 есть идеи почему? если нет - смотрите шпаргалку  
>testdb=> \c testdb postgres
Password for user postgres:
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "testdb" as user "postgres".
testdb=# grant SELECT on all TABLEs in SCHEMA testnm TO readonly;
GRANT
testdb=# \c testdb testread;
Password for user testread:
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "testdb" as user "testread".
testdb=> select * from testnm.t1;
 c1
----
  1
(1 row)
### 30 как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку  
### 31 сделайте select * from testnm.t1;  
### 32 получилось?  
да
### 33 есть идеи почему? если нет - смотрите шпаргалку  
>потому что ALTER default будет действовать для новых таблиц а grant SELECT on all TABLEs in SCHEMA testnm TO readonly отработал только для существующих на тот момент времени. надо сделать снова или grant SELECT или пересоздать таблицу
### 31 сделайте select * from testnm.t1;  
### 32 получилось?  
да
33 ура!  


### 34 теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);  
testdb=# \c testdb testread;
>Password for user testread:
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "testdb" as user "testread".
testdb=> create table t2(c1 integer);
CREATE TABLE
testdb=> insert into t2 values (2);
INSERT 0 1
testdb=> select * from t2;
 c
----
>  2
(1 row)

35 а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?  
36 есть идеи как убрать эти права? если нет - смотрите шпаргалку  
37 если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды  
38 теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);  
### 39 расскажите что получилось и почему
***Встречный вопрос: Поясните почему после отзыва прав на схему public и попытке создания таблицы t3 повторно права доступа не применились, с чем это связано? см код ниже:***
> testdb=> \c testdb postgres;
> Password for user postgres:  connection to server at "localhost" (::1), port 5432 failed: FATAL:  password authentication failed for user "postgres"  connection to server at "localhost" (::1), port 5432 failed: FATAL:  password authentication failed for user "postgres"
Previous connection kept
> testdb=> revoke CREATE on SCHEMA public FROM public;
WARNING:  no privileges could be revoked for "public"
REVOKE
testdb=> revoke all on DATABASE testdb FROM public;
WARNING:  no privileges could be revoked for "testdb"
REVOKE
testdb=> \c testdb testread;
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "testdb" as user "testread".
testdb=> create table t3(c1 integer);
CREATE TABLE
testdb=> insert into t3 values (2);
INSERT 0 1
testdb=> select * from t3;
 c1
----
  2
(1 row)
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzI3OTE0MTQ4LC01ODMxNDU4NTYsMTU3NT
gyMzg4Niw0NTk1MjEzNzYsMTM5NzY5Nzk4MiwtNzgyODI0Njk3
LDExMjM1MzI5ODEsLTQ1MzY2NjMxOCwtMTE1MzY5MDQxNV19
-->