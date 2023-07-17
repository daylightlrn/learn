# Домашнее задание к лекции "Резервное копирование и восстановление"
## Дата: 17.07.2023
## Тема: Резервное копирование
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ (2 core, 4 Gb, 20 Gb), ОС AstraLinux 1.7.4 mode 0, ПО Тантор (СУБД 15)

## Описание/Пошаговая инструкция выполнения домашнего задания:
_Создаем ВМ/докер c ПГ.
Создаем БД, схему и в ней таблицу.
Заполним таблицы автосгенерированными 100 записями.
Под линукс пользователем Postgres создадим каталог для бэкапов
Сделаем логический бэкап используя утилиту COPY
Восстановим во вторую таблицу данные из бэкапа.
Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц
Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!_


## Выполнение
_Создаем БД, схему и в ней таблицу._

## 1
```
postgres=# CREATE DATABASE backup1;
CREATE DATABASE
postgres=# \c backup1;
Вы подключены к базе данных "backup1" как пользователь "postgres".
backup1=# CREATE SCHEMA dbbackup;
CREATE SCHEMA
backup1=# CREATE TABLE dbbackup.tblbkp1(i int);
CREATE TABLE
```

_Заполним таблицы автосгенерированными 100 записями._

## 1
```
backup1=# INSERT INTO dbbackup.tblbkp1(i) SELECT GENERATE_SERIES(1, 100);
INSERT 0 100
backup1=# SELECT (i) FROM dbbackup.tblbkp1 limit 10;                                                              i
----
  1
  2
  3
  4
  5
  6
  7
  8
  9
 10
(10 строк)

backup1=# SELECT count (i) FROM dbbackup.tblbkp1;
 count
-------
   100
(1 строка)
```
_Под линукс пользователем Postgres создадим каталог для бэкапов_

## 1
```
backup1=# \! mkdir backup
```
_Сделаем логический бэкап используя утилиту COPY._

## 1
```
backup1=# COPY dbbackup.tblbkp1 TO '/var/lib/postgresql/backup/tblbackup.cvs' CSV HEADER;
COPY 100
```
_Восстановим во вторую таблицу данные из бэкапа._

## 1
```
backup1=# CREATE TABLE dbbackup.tblbkp2(i int);
CREATE TABLE
backup1=# COPY dbbackup.tblbkp2 FROM '/var/lib/postgresql/backup/tblbackup.cvs' CSV HEADER;
COPY 100
backup1=# SELECT (i) FROM dbbackup.tblbkp2 limit 10;
 i
----
  1
  2
  3
  4
  5
  6
  7
  8
  9
 10
(10 строк)
backup1=# SELECT count (i) FROM dbbackup.tblbkp2;
 count
-------
   100
(1 строка)
```
_Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц_

## 1
```
backup1=# \! pg_dump -d backup1 -t 'dbbackup.tblbkp1' -t 'dbbackup.tblbkp2' -Fc > /var/lib/postgresql/backup/tblbackup.gz
```
_Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!_

## 1
```
backup1=# CREATE DATABASE backup2;
CREATE DATABASE
backup1=# \c backup2;
Вы подключены к базе данных "backup2" как пользователь "postgres".
backup2=# CREATE SCHEMA dbbackup;
CREATE SCHEMA
backup2=# \! pg_restore --schema-only -d backup2 /var/lib/postgresql/backup/tblbackup.gz
backup2=# SELECT (i) FROM dbbackup.tblbkp1 limit 10;
 i
---
(0 строк)

backup2=# SELECT (i) FROM dbbackup.tblbkp2 limit 10;
 i
---
(0 строк)
backup2=# \! pg_restore --data-only -d backup2 --schema dbbackup -t tblbkp2 /var/lib/postgresql/backup/tblbackup.gz
backup2=# SELECT (i) FROM dbbackup.tblbkp2 limit 10;                                                              i
----
  1
  2
  3
  4
  5
  6
  7
  8
  9
 10
(10 строк)

backup2=# SELECT count (i) FROM dbbackup.tblbkp2;
 count
-------
   100
(1 строка)
backup2=# SELECT count (i) FROM dbbackup.tblbkp1;
 count
-------
     0
(1 строка)
```