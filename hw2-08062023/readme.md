# Домашнее задание к лекции "Установка PostgreSQL"
## Дата: 08.06.2023
## Тема: Работа с уровнями изоляции транзакции в PostgreSQL
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ (2 core, 8 Gb, 30 Gb), ОС AstraLinux 1.7.3 mode 0, ПО Тантор (СУБД 15, Платформа 2.01)
## Описание/Пошаговая инструкция выполнения домашнего задания:
_• создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом
• поставить на нем Docker Engine
• сделать каталог /var/lib/postgres
• развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql
• развернуть контейнер с клиентом postgres
• подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
• подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера
• удалить контейнер с сервером
• создать его заново
• подключится снова из контейнера с клиентом к контейнеру с сервером
• проверить, что данные остались на месте
• оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами_

## Выполнение
_Создаем две виртуальные машины, на первой размещаем контейнеры, вторая будет использоваться для удаленного доступа к контейнеру с СУБД. Устанавливаем на первую ВМ пакет Docker Engine. На второй ВМ устанавливаем клиент СУБД PostgreSQL._
### 1
```
root@postgres14:~# apt update
Пол:1 https://download.astralinux.ru/astra/stable/1.7_x86-64/repository-base 1.7_x86-64 InRelease [5 303 B]
Пол:2 https://download.astralinux.ru/astra/stable/1.7_x86-64/repository-extended 1.7_x86-64 InRelease [7 222 B]
Пол:3 https://download.astralinux.ru/astra/stable/1.7_x86-64/repository-base 1.7_x86-64/main amd64 Packages [4 124 kB]
...

root@postgres14:~# apt install docker.io -y
Чтение списков пакетов… Готово
Построение дерева зависимостей
Чтение информации о состоянии… Готово
...
```
_Создаем на первой ВМ каталог /var/lib/postgres. Готовим сеть для взаимодействия контейнеров. Загружаем и запускаем контейнер с СУБД PostgreSQL 15. Монтируем в него каталог /var/lib/postgres. Проверяем готовность контейнера и сети._

### 1
```
root@postgres14:~# mkdir -p /var/lib/postgres

root@postgres14:~# docker network create psql-net
87ccb7ef6b0cdb8b008260421733a50b97c2a30b7cda643ee920a224cb4f302f

root@postgres14:~# docker run --name psql-srv --network psql-net -e POSTGRES_PASSWORD='1!Password' -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
Unable to find image 'postgres:15' locally
15: Pulling from library/postgres
5b5fe70539cd: Pull complete
...

root@postgres14:~# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                    NAMES
b61819152865   postgres:15   "docker-entrypoint.s…"   6 seconds ago   Up 4 seconds   0.0.0.0:5432->5432/tcp   psql-srv

root@postgres14:~# docker network ls
NETWORK ID     NAME       DRIVER    SCOPE
6206df0eb08c   bridge     bridge    local
867f69344dff   host       host      local
628a65dc07a6   none       null      local
87ccb7ef6b0c   psql-net   bridge    local
```
_Разворачиваем контейнер с клиентом СУБД PostgreSQL. Подключаемся к контейнеру с СУБД. Создаем таблицу._

### 1
```
root@postgres14:~# docker run -it --rm --network psql-net --name psql-clnt postgres:15 psql -h psql-srv -U postgres
Password for user postgres:
psql (15.3 (Debian 15.3-1.pgdg120+1))
Type "help" for help.

postgres=# create table tsttbl(i int);
CREATE TABLE
postgres=# insert into tsttbl values(1);
INSERT 0 1
postgres=# insert into tsttbl values(2);
INSERT 0 1
postgres=# insert into tsttbl values(3);
INSERT 0 1
postgres=# select i from tsttbl;
 i
---
 1
 2
 3
(3 rows)

postgres=# \q
```
_На второй ВМ проверяем доступность сервиса СУБД. Со второй ВМ подключаемся к контейнеру с СУБД. Проверяем таблицу._
### 2

```
root@clonealse174cml:~# nmap -sT -p 5432 192.168.186.30
Starting Nmap 7.70 ( https://nmap.org ) at 2023-07-02 12:51 MSK
Nmap scan report for 192.168.186.30
Host is up (0.00097s latency).

PORT     STATE SERVICE
5432/tcp open  postgresql
MAC Address: 00:0C:29:CF:96:85 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.40 seconds
root@clonealse174cml:~#
root@clonealse174cml:~#
root@clonealse174cml:~# psql -h 192.168.186.30 -U postgres
Пароль пользователя postgres:
psql (14.5 (Debian 14.5-3+ci21), сервер 15.3 (Debian 15.3-1.pgdg120+1))
ПРЕДУПРЕЖДЕНИЕ: psql имеет базовую версию 14, а сервер - 15.
                Часть функций psql может не работать.
Введите "help", чтобы получить справку.

postgres=# select i from tsttbl;
 i
---
 1
 2
 3
(3 строки)

postgres=# \q
root@clonealse174cml:~#
```
_На первой ВМ удаляем контейнер с СУБД. Создаем контейнер заново. Подключаемся из контейнера с клиентом к контейнеру с СУБД и проверяем таблицу._

### 1  
```
root@postgres14:~# docker rm -f psql-srv
psql-srv
root@postgres14:~# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@postgres14:~#


root@postgres14:~# docker run --name psql-srv --network psql-net -e POSTGRES_PASSWORD='1!Password' -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
b5ecbcad991e1bb337b385dd68ac8ec288f767768d3a39df2397663914108c7a
root@postgres14:~#


root@postgres14:~# docker run -it --rm --network psql-net --name psql-clnt postgres:15 psql -h psql-srv -U postgres
Password for user postgres:
psql (15.3 (Debian 15.3-1.pgdg120+1))
Type "help" for help.

postgres=# select i from tsttbl;
 i
---
 1
 2
 3
(3 rows)

postgres=# \q
root@postgres14:~#
```