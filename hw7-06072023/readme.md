# Домашнее задание к лекции "Блокировки"
## Дата: 06.07.2023
## Тема: Журналы
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ (2 core, 4 Gb, 20 Gb), ОС AstraLinux 1.7.4 mode 0, ПО Тантор (СУБД 15)

## Описание/Пошаговая инструкция выполнения домашнего задания:
_Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.
Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.
Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
Задание со звездочкой*
Попробуйте воспроизвести такую ситуацию._


## Выполнение
_Настраиваем журнал для фиксации блокировок, удерживаемых более 200 мсек. Применяем конфигурацию._

## 1
```
postgres@alse174cmdt3:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# SELECT current_setting('log_lock_waits');
 current_setting
-----------------
 off
(1 строка)

postgres=# ALTER SYSTEM SET log_lock_waits = 'on';
ALTER SYSTEM
postgres=# SELECT name, setting, context, short_desc FROM pg_settings where name = 'log_lock_waits';
      name      | setting |  context  |                     short_desc
----------------+---------+-----------+----------------------------------------------------
 log_lock_waits | off     | superuser | Протоколировать длительные ожидания в блокировках.
(1 строка)

postgres=# SELECT current_setting('deadlock_timeout');
 current_setting
-----------------
 1s
(1 строка)

postgres=# ALTER SYSTEM SET deadlock_timeout = '200';
ALTER SYSTEM

postgres=# SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 строка)

postgres=# SELECT name, setting, context, short_desc FROM pg_settings where name = 'deadlock_timeout';
       name       | setting |  context  |                               short_desc
------------------+---------+-----------+------------------------------------------------------------------------
 deadlock_timeout | 200     | superuser | Задаёт интервал ожидания в блокировке до проверки на взаимоблокировку.
(1 строка)

postgres=#
```

_Готовим тестовую БД и таблицу. Подключаемся к БД. Наполняем таблицу._

## 1
```
postgres=# CREATE DATABASE test;
CREATE DATABASE
postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=# CREATE TABLE test (i integer);
CREATE TABLE
test=# INSERT INTO test VALUES (1), (2), (3);
INSERT 0 3
test=# SELECT i FROM test;
 i
---
 1
 2
 3
(3 строки)

test=#
```

_Готовим представление для просмотра блокировок._

## 1
```
test=# CREATE VIEW pg_locks AS
test-# SELECT pid,
test-#        locktype,
test-#        CASE locktype
test-#          WHEN 'relation' THEN relation::REGCLASS::text
test-#          WHEN 'virtualxid' THEN virtualxid::text
test-#          WHEN 'transactionid' THEN transactionid::text
test-#          WHEN 'tuple' THEN relation::REGCLASS::text||':'||tuple::text
test-#        END AS lockid,
test-#        mode,
test-#        granted
test-# FROM pg_locks;
CREATE VIEW
test=#
```

_Выполняем обновление одной и той же строки тремя командами UPDATE в трех разных сеансах_

## 1
```
postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=# BEGIN;
BEGIN
test=*# SELECT txid_current(), pg_backend_pid();
 txid_current | pg_backend_pid
--------------+----------------
          742 |           1375
(1 строка)

test=*# UPDATE test set i = 5 where i = 1;
UPDATE 1
```

## 2
```
postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=# BEGIN;
BEGIN
test=*# SELECT txid_current(), pg_backend_pid();
 txid_current | pg_backend_pid
--------------+----------------
          743 |           1377
(1 строка)

test=*# UPDATE test set i = 5 where i = 1;
UPDATE 1
```


## 3
```
postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=# BEGIN;
BEGIN
test=*# SELECT txid_current(), pg_backend_pid();
 txid_current | pg_backend_pid
--------------+----------------
          744 |           1378
(1 строка)

test=*# UPDATE test set i = 5 where i = 1;
UPDATE 1
```

_Посмотрим журнал на наличие данных о блокировоках._

## 4
```
postgres@alse174cmdt3:~$ cat /var/lib/postgresql/tantor-se-15/data/log/postgresql-2023-07-09_172749.log
2023-07-09 17:27:49.068 MSK [930] СООБЩЕНИЕ:  запускается PostgreSQL 15.2 on x86_64-pc-linux-gnu, compiled by gcc (AstraLinuxSE 8.3.0-6) 8.3.0, 64-bit
2023-07-09 17:27:49.069 MSK [930] СООБЩЕНИЕ:  для приёма подключений по адресу IPv4 "0.0.0.0" открыт порт 5432
2023-07-09 17:27:49.070 MSK [930] СООБЩЕНИЕ:  для приёма подключений по адресу IPv6 "::" открыт порт 5432
2023-07-09 17:27:49.073 MSK [930] СООБЩЕНИЕ:  для приёма подключений открыт Unix-сокет "/var/run/postgresql/.s.PGSQL.5432"
2023-07-09 17:27:49.091 MSK [957] СООБЩЕНИЕ:  система БД была выключена: 2023-07-09 17:27:27 MSK
2023-07-09 17:27:49.111 MSK [930] СООБЩЕНИЕ:  система БД готова принимать подключения
2023-07-09 17:32:49.188 MSK [955] СООБЩЕНИЕ:  начата контрольная точка: time
2023-07-09 17:32:49.193 MSK [955] СООБЩЕНИЕ:  контрольная точка завершена: записано буферов: 3 (0.0%); добавлено файлов WAL 0, удалено: 0, переработано: 0; запись=0.002 сек., синхр.=0.001 сек., всего=0.007 сек.; синхронизировано_файлов=2, самая_долгая_синхр.=0.001 сек., средняя=0.001 сек.; расстояние=0 kB, ожидалось=0 kB
2023-07-09 17:34:43.912 MSK [930] СООБЩЕНИЕ:  получен SIGHUP, файлы конфигурации перезагружаются
2023-07-09 17:34:43.916 MSK [930] СООБЩЕНИЕ:  параметр "log_lock_waits" принял значение "on"
2023-07-09 17:34:43.916 MSK [930] СООБЩЕНИЕ:  параметр "deadlock_timeout" принял значение "200"
2023-07-09 17:37:50.015 MSK [955] СООБЩЕНИЕ:  начата контрольная точка: time
2023-07-09 17:38:30.397 MSK [1351] ОШИБКА:  отношение "test" не существует (символ 8)
2023-07-09 17:38:30.397 MSK [1351] ОПЕРАТОР:  UPDATE test set i = 5 where i = 1;
2023-07-09 17:39:23.285 MSK [955] СООБЩЕНИЕ:  контрольная точка завершена: записано буферов: 925 (5.6%); добавлено файлов WAL 0, удалено: 0, переработано: 0; запись=93.256 сек., синхр.=0.004 сек., всего=93.270 сек.; синхронизировано_файлов=250, самая_долгая_синхр.=0.002 сек., средняя=0.001 сек.; расстояние=4243 kB, ожидалось=4243 kB
2023-07-09 17:40:02.188 MSK [1377] СООБЩЕНИЕ:  процесс 1377 продолжает ожидать в режиме ShareLock блокировку "transaction 742" в течение 200.531 мс
2023-07-09 17:40:02.188 MSK [1377] ПОДРОБНОСТИ:  Process holding the lock: 1375. Wait queue: 1377.
2023-07-09 17:40:02.188 MSK [1377] КОНТЕКСТ:  при изменении кортежа (0,1) в отношении "test"
2023-07-09 17:40:02.188 MSK [1377] ОПЕРАТОР:  UPDATE test set i = 5 where i = 1;
2023-07-09 17:40:23.343 MSK [1378] СООБЩЕНИЕ:  процесс 1378 продолжает ожидать в режиме ExclusiveLock блокировку "кортеж (0,1) отношения 16389 базы данных 16388" в течение 200.190 мс
2023-07-09 17:40:23.343 MSK [1378] ПОДРОБНОСТИ:  Process holding the lock: 1377. Wait queue: 1378.
2023-07-09 17:40:23.343 MSK [1378] ОПЕРАТОР:  UPDATE test set i = 5 where i = 1;
postgres@alse174cmdt3:~$
```

_Проверим блокировки для первой транзакции._

## 4
```
postgres=# SELECT * FROM pg_locks WHERE pid =1375;
   locktype    | database | relation | page | tuple | virtualxid | transactionid | classid | objid | objsubid | virtualtransaction | pid  |       mode       | granted | fastpath | wa
itstart
---------------+----------+----------+------+-------+------------+---------------+---------+-------+----------+--------------------+------+------------------+---------+----------+---
--------
 relation      |    16388 |    16389 |      |       |            |               |         |       |          | 6/35               | 1375 | RowExclusiveLock | t       | t        |
 virtualxid    |          |          |      |       | 6/35       |               |         |       |          | 6/35               | 1375 | ExclusiveLock    | t       | t        |
 transactionid |          |          |      |       |            |           742 |         |       |          | 6/35               | 1375 | ExclusiveLock    | t       | f        |
(3 строки)

postgres=# ;
```

_На изменяемое отношение ставится тип relation для test в режиме RowExclusiveLock. Типы virtualxid и transactionid в режиме ExclusiveLock каждая транзакция держит для себя._

## 4
```
postgres=# SELECT * FROM pg_locks WHERE pid =1377;
   locktype    | database | relation | page | tuple | virtualxid | transactionid | classid | objid | objsubid | virtualtransaction | pid  |       mode       | granted | fastpath |
     waitstart
---------------+----------+----------+------+-------+------------+---------------+---------+-------+----------+--------------------+------+------------------+---------+----------+------
-------------------------
 relation      |    16388 |    16389 |      |       |            |               |         |       |          | 3/16               | 1377 | RowExclusiveLock | t       | t        |
 virtualxid    |          |          |      |       | 3/16       |               |         |       |          | 3/16               | 1377 | ExclusiveLock    | t       | t        |
 tuple         |    16388 |    16389 |    0 |     1 |            |               |         |       |          | 3/16               | 1377 | ExclusiveLock    | t       | f        |
 transactionid |          |          |      |       |            |           743 |         |       |          | 3/16               | 1377 | ExclusiveLock    | t       | f        |
 transactionid |          |          |      |       |            |           742 |         |       |          | 3/16               | 1377 | ShareLock        | f       | f        | 2023-
07-09 17:40:01.987452+03
(5 строк)

postgres=#
```


_Транзакция ждет блокировки transactionid в режиме ShareLock для первой транзакции. Блокировка tuple для строки, которая обновляется._

## 4
```
postgres=# SELECT * FROM pg_locks WHERE pid =1378;
   locktype    | database | relation | page | tuple | virtualxid | transactionid | classid | objid | objsubid | virtualtransaction | pid  |       mode       | granted | fastpath |
      waitstart
---------------+----------+----------+------+-------+------------+---------------+---------+-------+----------+--------------------+------+------------------+---------+----------+-----
--------------------------
 relation      |    16388 |    16389 |      |       |            |               |         |       |          | 4/3                | 1378 | RowExclusiveLock | t       | t        |
 virtualxid    |          |          |      |       | 4/3        |               |         |       |          | 4/3                | 1378 | ExclusiveLock    | t       | t        |
 transactionid |          |          |      |       |            |           744 |         |       |          | 4/3                | 1378 | ExclusiveLock    | t       | f        |
 tuple         |    16388 |    16389 |    0 |     1 |            |               |         |       |          | 4/3                | 1378 | ExclusiveLock    | f       | f        | 2023
-07-09 17:40:23.143545+03
(4 строки)

postgres=#
```

_Транзакция в процессе получения блокировки tuple для строки, которую обновляет._

## 4
```
postgres=# SELECT pid, wait_event_type, wait_event, pg_blocking_pids(pid) FROM pg_stat_activity WHERE backend_type = 'client backend';
 pid  | wait_event_type |  wait_event   | pg_blocking_pids
------+-----------------+---------------+------------------
 1377 | Lock            | transactionid | {1375}
 1378 | Lock            | tuple         | {1377}
 1420 |                 |               | {}
 1375 | Client          | ClientRead    | {}
(4 строки)

postgres=#
```

_Откатим назад транзакцию во всех сеансах._

## 1
```
test=*# ROLLBACK;
ROLLBACK
test=#
```

## 2
```
test=*# ROLLBACK;
ROLLBACK
test=#
```

## 3
```
test=*# ROLLBACK;
ROLLBACK
test=#
```

_Моделируем взаимоблокировку трех транзакций._

## 1
```
test=# BEGIN;
BEGIN
test=*# UPDATE test set i = 1 WHERE i = 1;
UPDATE 1
```

__

## 2
```
test=# BEGIN;
BEGIN
test=*# UPDATE test set i = 1 WHERE i = 2;
UPDATE 1

```

__

## 3
```
test=# BEGIN;
BEGIN
test=*# UPDATE test set i = 1 WHERE i = 3;
UPDATE 1
```

## 1
```
test=*# UPDATE test set i = 1 WHERE i = 2;
UPDATE 1
```

__

## 2
```
test=*# UPDATE test set i = 1 WHERE i = 3;
UPDATE 1
```

__

## 3
```
test=*# UPDATE test set i = 1 WHERE i = 1;
ОШИБКА:  обнаружена взаимоблокировка
ПОДРОБНОСТИ:  Процесс 1378 ожидает в режиме ShareLock блокировку "transaction 749"; заблокирован процессом 1375.
Процесс 1375 ожидает в режиме ShareLock блокировку "transaction 750"; заблокирован процессом 1377.
Процесс 1377 ожидает в режиме ShareLock блокировку "transaction 751"; заблокирован процессом 1378.
ПОДСКАЗКА:  Подробности запроса смотрите в протоколе сервера.
КОНТЕКСТ:  при изменении кортежа (0,1) в отношении "test"
```

_Получили сообщение о взаимоблокировке. То же в журнале._

## 4
```
2023-07-09 18:00:11.804 MSK [1378] ОШИБКА:  обнаружена взаимоблокировка
2023-07-09 18:00:11.804 MSK [1378] ПОДРОБНОСТИ:  Процесс 1378 ожидает в режиме ShareLock блокировку "transaction 749"; заблокирован процессом 1375.
        Процесс 1375 ожидает в режиме ShareLock блокировку "transaction 750"; заблокирован процессом 1377.
        Процесс 1377 ожидает в режиме ShareLock блокировку "transaction 751"; заблокирован процессом 1378.
        Процесс 1378: UPDATE test set i = 1 WHERE i = 1;
        Процесс 1375: UPDATE test set i = 1 WHERE i = 2;
        Процесс 1377: UPDATE test set i = 1 WHERE i = 3;
2023-07-09 18:00:11.804 MSK [1378] ПОДСКАЗКА:  Подробности запроса смотрите в протоколе сервера.
2023-07-09 18:00:11.804 MSK [1378] КОНТЕКСТ:  при изменении кортежа (0,1) в отношении "test"
2023-07-09 18:00:11.804 MSK [1378] ОПЕРАТОР:  UPDATE test set i = 1 WHERE i = 1;
2023-07-09 18:00:11.806 MSK [1377] СООБЩЕНИЕ:  процесс 1377 получил в режиме ShareLock блокировку "transaction 751" через 7461.272 мс
2023-07-09 18:00:11.806 MSK [1377] КОНТЕКСТ:  при изменении кортежа (0,3) в отношении "test"
2023-07-09 18:00:11.806 MSK [1377] ОПЕРАТОР:  UPDATE test set i = 1 WHERE i = 3;
```

_Выполним откат во всех трех сеансах._

## 1
```
test=*# ROLLBACK;
ROLLBACK
test=#
```

__

## 2
```
test=*# ROLLBACK;
ROLLBACK
test=#
```

## 3
```
test=*# ROLLBACK;
ROLLBACK
test=#
```
_Две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), могут заблокировать друг друга если будут обновлять строки в разном порядке, с разными планами выполнения (первая будет выполнять по индексу, а вторая последовательно)._