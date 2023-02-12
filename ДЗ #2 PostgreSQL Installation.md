#### Создание контейнеров Docker 
- Создаем docker-сеть: $ _sudo docker network create pg-net_
- Развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres:  $ _docker pull postgres:14_ 
- Подключаем созданную сеть к контейнеру сервера Postgres:
 $ _sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14_
 - Развернуть контейнер с клиентом postgres (Запускаем отдельный контейнер с клиентом в общей сети с БД):
 $ _sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-docker -U postgres PGPASSWORD=postgres psql -U postgres_
- Проверяем, что подключились через отдельный контейнер|  -- выводит на экран список всех запущенных контейнеров.
 $ _sudo docker ps -a_
***(Появился список контейнеров с установленным PostgreSQL 14  )***
- Подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
_postgres#
create table persons(id serial, first_name text, second_name text); 
insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
insert into persons(first_name, second_name) values('petr', 'petrov');_

- подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера
***== Установка клиентской части psql на своем ПК (WSL2) и ВМ***
_sudo apt install postgresql-client-common_
_sudo apt-get install postgresql-client_

***== Проверяем с ВМ локально (удачно)***
_psql -h localhost -U postgres -d postgres
postgres#
select * from persons;_ ***(видим 2 строчки)***

***== Проверяем с ПК из Ubuntu WSL2 (удачно)***
psql -p 5432 -U postgres -h 158.160.27.72 -d postgres -W
postgres#
select * from persons;_ ***(видим 2 строчки)***

- Удалить контейнер с сервером.
_$ sudo docker ps -a_
_$ docker stop b58cae3343f5_
_$ docker rm b58cae3343f5_

***Удаление получилось только после остановки. Так же попробовал  выполнить $ docker rm $(docker ps -a -q -f status=exited) - зачистка всех контейнеров)***

- Создать его заново
$ _sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14_
- Подключится снова из контейнера с клиентом к контейнеру с сервером
_sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-docker -U postgres_
- Проверить, что данные остались на месте
_postgres#
select * from persons;_
- То же самое с ПК из Ubuntu WSL2:
_psql -p 5432 -U postgres -h 158.160.27.72 -d postgres -W_
***(видим те 2 строчки)***
