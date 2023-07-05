# Домашнее задание к лекции "Логический уровень PostgreSQL"
## Дата: 22.06.2023
## Тема: Логический уровень psql и системный каталог
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ (2 core, 4 Gb, 20 Gb), ОС AstraLinux 1.7.4 mode 0, ПО Тантор (СУБД 15)

## Описание/Пошаговая инструкция выполнения домашнего задания:
_1 создайте новый кластер PostgresSQL 14
2 зайдите в созданный кластер под пользователем postgres
3 создайте новую базу данных testdb 
4 зайдите в созданную базу данных под пользователем postgres
5 создайте новую схему testnm
6 создайте новую таблицу t1 с одной колонкой c1 типа integer
7 вставьте строку со значением c1=1
8 создайте новую роль readonly
9 дайте новой роли право на подключение к базе данных testdb
10 дайте новой роли право на использование схемы testnm
11 дайте новой роли право на select для всех таблиц схемы testnm
12 создайте пользователя testread с паролем test123
13 дайте роль readonly пользователю testread
14 зайдите под пользователем testread в базу данных testdb <<<
15 сделайте select * from t1; <<<
16 получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
17 напишите что именно произошло в тексте домашнего задания
18 у вас есть идеи почему? ведь права то дали?
19 посмотрите на список таблиц
20 подсказка в шпаргалке под пунктом 20
21 а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
22 вернитесь в базу данных testdb под пользователем postgres
23 удалите таблицу t1
24 создайте ее заново но уже с явным указанием имени схемы testnm
25 вставьте строку со значением c1=1
26 зайдите под пользователем testread в базу данных testdb <<<
27 сделайте select * from testnm.t1; <<<
28 получилось?
29 есть идеи почему? если нет - смотрите шпаргалку
30 как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
31 сделайте select * from testnm.t1;
32 получилось?
33 есть идеи почему? если нет - смотрите шпаргалку
31 сделайте select * from testnm.t1;
32 получилось?
33 ура!
34 теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
35 а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
36 есть идеи как убрать эти права? если нет - смотрите шпаргалку
37 если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды
38 теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
39 расскажите что получилось и почему_

## Выполнение
_Используем виртуальную машину из предыдущего задания. Проверяем статус СУБД. запускам СУБД. Проверяем возможность сетевого подключения. Подключаемся пользователем postgres._

## 1
```
postgres@alse174cmdfdt2:~$ pg_ctl status
pg_ctl: сервер не работает
postgres@alse174cmdfdt2:~$ pg_ctl start
ожидание запуска сервера....2023-07-05 11:53:55.536 MSK [1057] СООБЩЕНИЕ:  запускается PostgreSQL 15.2 on x86_64-pc-linux-gnu, compiled by gcc (AstraLinuxSE 8.3.0-6) 8.3.0, 64-bit
2023-07-05 11:53:55.537 MSK [1057] СООБЩЕНИЕ:  для приёма подключений по адресу IPv4 "0.0.0.0" открыт порт 5432
2023-07-05 11:53:55.537 MSK [1057] СООБЩЕНИЕ:  для приёма подключений по адресу IPv6 "::" открыт порт 5432
2023-07-05 11:53:55.540 MSK [1057] СООБЩЕНИЕ:  для приёма подключений открыт Unix-сокет "/var/run/postgresql/.s.PGSQL.5432"
2023-07-05 11:53:55.553 MSK [1060] СООБЩЕНИЕ:  система БД была выключена: 2023-07-05 11:53:50 MSK
2023-07-05 11:53:55.568 MSK [1057] СООБЩЕНИЕ:  система БД готова принимать подключения
 готово
сервер запущен
postgres@alse174cmdfdt2:~$ nmap -sT -p 5432 192.168.186.172
Starting Nmap 7.70 ( https://nmap.org ) at 2023-07-05 11:54 MSK
Nmap scan report for alse174cmdfdt2.example.dn (192.168.186.172)
Host is up (0.00013s latency).

2023-07-05 11:54:15.631 MSK [1065] СООБЩЕНИЕ:  не удалось получить данные от клиента: Соединение разорвано другой стороной
Host is up (0.00017s latency).

PORT     STATE SERVICE
5432/tcp open  postgresql
```

_Создаем БД testdb. Подключаемся к ней пользователем postgres. Создаем схему testnm. Создаем таблицу t1. Вставляем строку в таблицу t1. Создаем новую роль readonly. Предоставляем роли readonly право на подключение к базе данных testdb. Даем роли readonly право на использование схемы testnm. Даем новой роли право на select для всех таблиц схемы testnm. Создаем пользователя testread с паролем test123. Даем роль readonly пользователю testread._

```
Nmap done: 1 IP address (1 host up) scanned in 0.09 seconds
postgres@alse174cmdfdt2:~$ psql -h 192.168.186.172 -d postgres -U postgres -W
Пароль:
SET
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# CREATE DATABASE testdb;
CREATE DATABASE
postgres=# \c testdb
Пароль:
Вы подключены к базе данных "testdb" как пользователь "postgres".
testdb=# CREATE SCHEMA testnm;
CREATE SCHEMA
testdb=# CREATE TABLE t1(c1 int);
CREATE TABLE
testdb=# INSERT INTO t1 values(1);
INSERT 0 1
testdb=# CREATE ROLE readonly;
CREATE ROLE
testdb=# GRANT CONNECT ON DATABASE testdb TO readonly;
GRANT
testdb=# GRANT USAGE ON SCHEMA testnm TO readonly;
GRANT
testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
GRANT
testdb=# CREATE USER testread WITH PASSWORD 'test123';
CREATE ROLE
testdb=# GRANT readonly TO testread;
GRANT ROLE
testdb=# \q
postgres@alse174cmdfdt2:~$
```

_Входим под пользователем testread в базу данных testdb. Выполняем выборку из таблицы t1.
Запрос не выполнен, так как у нас право на выполнение select только для схемы testnm  
В переменной окружения search_path указано $user, public при том что схемы $user нет, то таблица по умолчанию создалась в public_
```
postgres@alse174cmdfdt2:~$ psql -h 192.168.186.172 -d testdb -U testread -W
Пароль:
SET
psql (15.2)
Введите "help", чтобы получить справку.

testdb=> SELECT * FROM t1;
2023-07-05 13:17:56.171 MSK [1015] ОШИБКА:  нет доступа к таблице t1
2023-07-05 13:17:56.171 MSK [1015] ОПЕРАТОР:  SELECT * FROM t1;
ОШИБКА:  нет доступа к таблице t1
```

_Запрос выдает ошибку, так как у нас право на выполнение select только для схемы testnm. В переменной окружения search_path указано $user, public при том что схемы $user нет, то таблица по умолчанию создалась в public.
```
testdb=> \dt
         Список отношений
 Схема  | Имя |   Тип   | Владелец
--------+-----+---------+----------
 public | t1  | таблица | postgres
(1 строка)

testdb=>
testdb=> \q
```
_Вернемся в базу данных testdb под пользователем postgres. Удалим таблицу t1. Создаем таблицу, указав имя схемы testnm. Вставим строку со значением c1=1._
```
postgres@alse174cmdfdt2:~$ psql -h 192.168.186.172 -d testdb -U postgres -W
Пароль:
SET
psql (15.2)
Введите "help", чтобы получить справку.

testdb=# DROP TABLE t1;
DROP TABLE
testdb=# CREATE TABLE testnm.t1(c1 int);
CREATE TABLE
testdb=# \dt testnm.*
         Список отношений
 Схема  | Имя |   Тип   | Владелец
--------+-----+---------+----------
 testnm | t1  | таблица | postgres
(1 строка)

testdb=# INSERT INTO testnm.t1 values(1);
INSERT 0 1
testdb=# \q
```
_Входим под пользователем testread в базу данных testdb. Выполняем запрос на выборку.
Запрос выдал ошибку, так как **GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;** дал доступ **только** для существующих на тот момент таблиц, а t1 пересоздавалась._
```
postgres@alse174cmdfdt2:~$ psql -h 192.168.186.172 -d testdb -U testread -W
Пароль:
SET
psql (15.2)
Введите "help", чтобы получить справку.

testdb=> SELECT * FROM testnm.t1;
2023-07-05 13:40:42.516 MSK [1137] ОШИБКА:  нет доступа к таблице t1
2023-07-05 13:40:42.516 MSK [1137] ОПЕРАТОР:  SELECT * FROM testnm.t1;
ОШИБКА:  нет доступа к таблице t1
testdb=> \q
```
_Подключимся к БД testdb под пользователем postgres. Выдаем роли readonly право на select для всех таблиц схемы testnm. Запрос на выборку работает._
```
postgres@alse174cmdfdt2:~$ psql -h 192.168.186.172 -d testdb -U postgres -W
Пароль:
SET
psql (15.2)
Введите "help", чтобы получить справку.

testdb=# ALTER DEFAULT PRIVILEGES IN SCHEMA testnm GRANT SELECT ON TABLES to readonly;
ALTER DEFAULT PRIVILEGES
testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
GRANT
testdb=# \q
postgres@alse174cmdfdt2:~$ psql -h 192.168.186.172 -d testdb -U testread -W
Пароль:
SET
psql (15.2)
Введите "help", чтобы получить справку.

testdb=> SELECT * FROM testnm.t1;
 c1
----
  1
(1 строка)
```
_ALTER DEFAULT работает для новых таблиц. GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly; работает только для существующих на тот момент времени  
Необходимо было сделать снова GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly; или создать таблицу заново._

```
testdb=> CREATE TABLE t2(c1 integer);
2023-07-05 13:43:47.004 MSK [1147] ОШИБКА:  нет доступа к схеме public (символ 14)
2023-07-05 13:43:47.004 MSK [1147] ОПЕРАТОР:  CREATE TABLE t2(c1 integer);
ОШИБКА:  нет доступа к схеме public
СТРОКА 1: CREATE TABLE t2(c1 integer);
                       ^
testdb=> \q
postgres@alse174cmdfdt2:~$ psql -h 192.168.186.172 -d testdb -U postgres -W
Пароль:
SET
psql (15.2)
Введите "help", чтобы получить справку.

testdb=# select * from pg_namespace;
  oid  |      nspname       | nspowner |                            nspacl
-------+--------------------+----------+---------------------------------------------------------------
    99 | pg_toast           |       10 |
    11 | pg_catalog         |       10 | {postgres=UC/postgres,=U/postgres}
  2200 | public             |     6171 | {pg_database_owner=UC/pg_database_owner,=U/pg_database_owner}
 13140 | information_schema |       10 | {postgres=UC/postgres,=U/postgres}
 16429 | testnm             |       10 | {postgres=UC/postgres,readonly=U/postgres}
(5 строк)

testdb=# show search_path;
   search_path
-----------------
 "$user", public
(1 строка)

testdb=# \dn
        Список схем
  Имя   |     Владелец
--------+-------------------
 public | pg_database_owner
 testnm | postgres
(2 строки)

testdb=# \dp testnm.*
                                   Права доступа
 Схема  | Имя |   Тип   |       Права доступа       | Права для столбцов | Политики
--------+-----+---------+---------------------------+--------------------+----------
 testnm | t1  | таблица | postgres=arwdDxt/postgres+|                    |
        |     |         | readonly=r/postgres       |                    |
(1 строка)

testdb=# \dp public.*
                           Права доступа
 Схема | Имя | Тип | Права доступа | Права для столбцов | Политики
-------+-----+-----+---------------+--------------------+----------
(0 строк)

testdb=#
```
_Попытка создания таблицы t2 в версии СУБД 15 завершится ошибкой, поскольку в сентябре 2021 года был зафиксирован патч для версии PostgreSQL 15, который вносит заметное изменение для пользователей: привилегия CREATE для публичной схемы больше не устанавливается по умолчанию. Это рекомендация CVE-2018-1058 (https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-1058)._