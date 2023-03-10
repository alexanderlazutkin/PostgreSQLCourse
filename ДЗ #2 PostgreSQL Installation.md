##  Домашнее задание 2
###  Установка PostgreSQL

оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами

> Предисловие: Указанный сценарий выстрадан с 5-6 попытки без ошибок как слепой котенок делал не осознавая что. Пока не нашел в интернете основы работы с докером и примеры развертывания в нем PostgreSQL решить ДЗ не представлялось возможным.

  

##### Создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом и поставить на нем Docker Engine
> Сделано в Yandex Cloud :
> _yc compute instance create --name pg-instance --hostname pg-instance --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --metadata-from-file ssh-keys=C:/Users/User/.ssh/sshkeys.txt_

  

++++++++++++++ Terminal PowerShell ++++++++++++++++

>ssh-agent
>ssh user@158.160.27.72

#### Установка и настройка по документации ttps://docs.docker.com/engine/install/ubuntu/
#####  Set up the repository:

>sudo apt-get update
>sudo apt-get install \\
	ca-certificates \\
	curl \\
	gnupg \\
	lsb-release

#####  Add Docker’s official GPG key:
>sudo mkdir -m 0755 -p /etc/apt/keyrings

>curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

#####  Use the following command to set up the repository: :

>echo \\
>"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] >https://download.docker.com/linux/ubuntu \\
>$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

  

#####  Install Docker Engine
- Update the apt package index:
>sudo apt-get update
- Install Docker Engine, containerd, and Docker Compose (lastest verion)
>sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

- Verify that the Docker Engine installation is successful by running the hello-world image:
>sudo docker run hello-world

***(Успешно отработало)***
***--Альтернативно указано в курсе (но мне пока непонятно и я так не делал):*** curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER

  
#### Подготовка к созданию контейнеров Docker

- Сделать каталог /var/lib/postgres
>sudo mkdir /var/lib/postgresql

>sudo mkdir /var/lib/postgresql/data

***(Пришлось выполнить эти две инструкции иначе пункт "Подключаем созданную сеть к контейнеру сервера Postgres: " не работал)***

  

#####  Run docker as non-root user then you need to add it to the docker group. https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user

- Create the docker group if it does not exist: 
> sudo groupadd docker
- Add your user to the docker group:
>sudo usermod -aG docker $USER
- Log in to the new docker group (to avoid having to log out / log in again; but if not enough, try to reboot):
>newgrp docker
- Check if docker can be run without root: 
>docker run hello-world

- Reboot if still got error: $ _reboot_ ***(Пропустил, т.к. отработало без ошибок)***

#### Создание контейнеров Docker 
##### Создаем docker-сеть: $ _sudo docker network create pg-net_
- Развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres:  
>docker pull postgres:14
- Подключаем созданную сеть к контейнеру сервера Postgres:
>sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
 - Развернуть контейнер с клиентом postgres (Запускаем отдельный контейнер с клиентом в общей сети с БД):
>sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-docker -U postgres PGPASSWORD=postgres psql -U postgres
- Проверяем, что подключились через отдельный контейнер|  -- выводит на экран список всех запущенных контейнеров.
>sudo docker ps -a

***(Появился список контейнеров с установленным PostgreSQL 14  )***
- Подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
>postgres#
>create table persons(id serial, first_name text, second_name text); 
>insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
>insert into persons(first_name, second_name) values('petr', 'petrov');

- Подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера

***Установка клиентской части psql на своем ПК (WSL2) и ВМ***
>sudo apt install postgresql-client-common
>sudo apt-get install postgresql-client

***Проверяем с ВМ локально (удачно)***
>psql -h localhost -U postgres -d postgres
>postgres#
>select * from persons;

***(видим 2 строчки)***

***== Проверяем с ПК из Ubuntu WSL2 (удачно)***
>psql -p 5432 -U postgres -h 158.160.27.72 -d postgres -W
>postgres#
>select * from persons;

***(видим 2 строчки)***

- Удалить контейнер с сервером.
>sudo docker ps -a
>docker stop b58cae3343f5
>docker rm b58cae3343f5

***Удаление получилось только после остановки. Так же попробовал  выполнить $ docker rm $(docker ps -a -q -f status=exited) - зачистка всех контейнеров)***

- Создать контейнер заново
>sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
- Подключится снова из контейнера с клиентом к контейнеру с сервером
>sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-docker -U postgres
- Проверить, что данные остались на месте
>postgres#
>select * from persons;

***(видим 2 строчки)***
- То же самое с ПК из Ubuntu WSL2:
>psql -p 5432 -U postgres -h 158.160.27.72 -d postgres -W

***(видим те 2 строчки)***
<!--stackedit_data:
eyJoaXN0b3J5IjpbODU0MjMzNjE2XX0=
-->