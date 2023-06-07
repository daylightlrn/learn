# Домашнее задание к лекции "SQL и реляционные СУБД. Введение в PostgreSQL"
## Дата: 05.06.2023
## Тема: Работа с уровнями изоляции транзакции в PostgreSQL
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ (2 core, 8 Gb, 30 Gb), ОС AstraLinux 1.7.3 mode 0, ПО Тантор (СУБД 15, Платформа 2.01)
## Описание/Пошаговая инструкция выполнения домашнего задания:
_создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере далее создать инстанс виртуальной машины с дефолтными параметрами добавить свой ssh ключ в metadata ВМ поставить PostgreSQL зайти вторым ssh (вторая сессия) запустить везде psql из под пользователя postgres_

## Выполнение
### 1
_выключить auto commit
сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
посмотреть текущий уровень изоляции: show transaction isolation level
начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');_
### 1
```
postgres=# \set AUTOCOMMIT OFF
postgres=# create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
CREATE TABLE
INSERT 0 1
INSERT 0 1
COMMIT
postgres=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 строка)
```
### 2
```
postgres=# \set AUTOCOMMIT OFF
postgres=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 строка)
```
_начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');_
### 1
```
insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
```
_сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?_
### 2
```
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 строки)
```
строки нет, транзакция не завершена
### 1
_завершить первую транзакцию - commit;_
```
postgres=*# commit;
COMMIT
```
_сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?_
### 2  
```
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 строки)
```
_завершите транзакцию во второй сессии_
### 2
```
postgres=*# commit;
COMMIT
```
**Ответ:** в режиме по-умолчанию ```read committed``` в своей транзакции видим "```неповторяемое чтение```" только завершенные другие транзакции.

_начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');_
### 1
```
postgres=# set transaction isolation level repeatable read;
SET
postgres=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
```
_сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?_
### 2
```
postgres=# set transaction isolation level repeatable read;
SET
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 строки)
```
_завершить первую транзакцию - commit;_
### 1
```
postgres=*# commit;
COMMIT
```
_сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?_
### 2
```
otus=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
_завершить вторую транзакцию
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?_
### 2
```
postgres=*# commit;
COMMIT
postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
(4 строки)
```
**Ответ:** в режиме ```repeatable read``` не видно "```неповторяемое чтение```", в выбранной транзакции выводятся зафиксированные до начала транзакции данные.
