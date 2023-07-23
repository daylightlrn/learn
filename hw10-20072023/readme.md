# Домашнее задание к лекции "Виды и устройство репликации в PostgreSQL"
## Дата: 20.07.2023
## Тема: Репликация
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ
alse174cmdt3, IP 192.168.186.173,
alse174cmdt4, IP 192.168.186.174,
alse174cmdt5, IP 192.168.186.175,
alse174cmdt6, IP 192.168.186.176,
экраны 1 - 4, (2 core, 4 Gb, 20 Gb), ОС AstraLinux 1.7.4 mode 0, ПО Тантор (СУБД 15)

## Описание/Пошаговая инструкция выполнения домашнего задания:
_На 1 ВМ создаем таблицы test1 для записи, test2 для запросов на чтение.
Создаем публикацию таблицы test1 и подписываемся на публикацию таблицы test2 с ВМ №2.
На 2 ВМ создаем таблицы test2 для записи, test1 для запросов на чтение.
Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.
3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).
ДЗ сдается в виде миниотчета на гитхабе с описанием шагов и с какими проблемами столкнулись.
Задание со * реализовать горячее реплицирование для высокой доступности на ВМ №4.
Источником должна выступать ВМ №3._


## Выполнение
_На трех ВМ (alse174cmdt3 - alse174cmdt5) расширяем уровень журналов для логической репликации. Создаем тестовые БД и таблицы. Перезагружаем инстансы._

_Выпоняем настройки на первой ВМ alse174cmdt3 IP 192.168.186.173_
## 1
```
postgres@alse174cmdt3:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# ALTER SYSTEM SET wal_level = logical;
ALTER SYSTEM
postgres=# CREATE DATABASE test;
CREATE DATABASE
postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=# CREATE TABLE test1 (i int, row varchar(10));
CREATE TABLE
test=# INSERT INTO test1 VALUES(1, 'Значение1');
INSERT 0 1
test=# INSERT INTO test1 VALUES(2, 'Значение2');
INSERT 0 1
test=# INSERT INTO test1 VALUES(3, 'Значение3');
INSERT 0 1
test=# SELECT i, row FROM test1;
 i |    row
---+-----------
 1 | Значение1
 2 | Значение2
 3 | Значение3
(3 строки)

test=# \q
postgres@alse174cmdt3:~$ pg_ctl stop
ожидание завершения работы сервера.... готово
сервер остановлен
```
_На первой ВМ alse174cmdt3 IP 192.168.186.173 создаем и проверяем публикацию_
## 1
```
postgres@alse174cmdt3:~$ pg_ctl start
ожидание запуска сервера....2023-07-23 13:13:20.181 MSK [1314] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
2023-07-23 13:13:20.181 MSK [1314] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
 готово
сервер запущен
postgres@alse174cmdt3:~$ pg_ctl status
pg_ctl: сервер работает (PID: 1314)
/opt/tantor/db/15/bin/postgres
postgres@alse174cmdt3:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=# CREATE PUBLICATION public_test1 FOR TABLE test1;
CREATE PUBLICATION

test=# \dRp+
                                 Публикация public_test1
 Владелец | Все таблицы | Добавления | Изменения | Удаления | Опустошения | Через корень
----------+-------------+------------+-----------+----------+-------------+--------------
 postgres | f           | t          | t         | t        | t           | f
Таблицы:
    "public.test1"

test=#
```

_Выполняем настройку второй ВМ alse174cmdt4 IP 192.168.186.174_

## 2
```
postgres@alse174cmdt4:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# ALTER SYSTEM SET wal_level = logical;
ALTER SYSTEM
postgres=# CREATE DATABASE test;
CREATE DATABASE
postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=# CREATE TABLE test2 (i int, row varchar(10));
CREATE TABLE
test=# INSERT INTO test2 VALUES(1, 'Значение3');
INSERT 0 1
test=# INSERT INTO test2 VALUES(2, 'Значение4');
INSERT 0 1
test=# INSERT INTO test2 VALUES(3, 'Значение5');
INSERT 0 1
test=# SELECT i, row FROM test2;
 i |    row
---+-----------
 1 | Значение3
 2 | Значение4
 3 | Значение5
(3 строки)

test=# \q
postgres@alse174cmdt4:~$ pg_ctl stop
ожидание завершения работы сервера.... готово
сервер остановлен
```

__На второй ВМ alse174cmdt4 IP 192.168.186.174 создаем и проверяем публикацию_

## 2
```
postgres@alse174cmdt4:~$ pg_ctl start
ожидание запуска сервера....2023-07-23 13:21:57.822 MSK [1589] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
2023-07-23 13:21:57.822 MSK [1589] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
 готово
сервер запущен
postgres@alse174cmdt4:~$ pg_ctl status
pg_ctl: сервер работает (PID: 1589)
/opt/tantor/db/15/bin/postgres
postgres@alse174cmdt4:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=# CREATE PUBLICATION public_test2 FOR TABLE test2;
CREATE PUBLICATION
test=# \dRp+
                                 Публикация public_test2
 Владелец | Все таблицы | Добавления | Изменения | Удаления | Опустошения | Через корень
----------+-------------+------------+-----------+----------+-------------+--------------
 postgres | f           | t          | t         | t        | t           | f
Таблицы:
    "public.test2"

test=#
```

_Создаем подписку на первой ВМ alse174cmdt3 со второй ВМ alse174cmdt4 IP 192.168.186.174. Проверяем состояние подписки. Делаем запросы к таблицам._

## 1
```
test=# CREATE TABLE test2 (i int,row varchar(10));
CREATE TABLE
test=# CREATE SUBSCRIPTION subscr_test2
test-# CONNECTION 'host=192.168.186.174 port=5432 user=postgres password=postgres dbname=test'
test-# PUBLICATION public_test2 WITH (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "subscr_test2"
CREATE SUBSCRIPTION
test=# \dRs
                  Список подписок
     Имя      | Владелец | Включён |   Публикация
--------------+----------+---------+----------------
 subscr_test2 | postgres | t       | {public_test2}
(1 строка)

test=# SELECT i, row FROM test2;
 i |    row
---+-----------
 1 | Значение3
 1 | Значение4
 1 | Значение5
(3 строки)

test=# SELECT i, row FROM test1;
 i |    row
---+-----------
 1 | Значение1
 2 | Значение2
 3 | Значение3
(3 строки)

test=# SELECT * FROM pg_stat_subscription \gx
-[ RECORD 1 ]---------+------------------------------
subid                 | 16398
subname               | subscr_test2
pid                   | 1528
relid                 |
received_lsn          | 0/19B11C0
last_msg_send_time    | 2023-07-23 13:28:08.216298+03
last_msg_receipt_time | 2023-07-23 13:28:08.217061+03
latest_end_lsn        | 0/19B11C0
latest_end_time       | 2023-07-23 13:28:08.216298+03
```
_Подписка на публикацию таблицы test2 со второй ВМ выполнена._

_Создаем подписку на второй ВМ alse174cmdt4 с первой ВМ alse174cmdt3 IP 192.168.186.173. Проверяем состояние подписки. Делаем запосы к таблицам._

## 2
```
test=# CREATE TABLE test1 (i int,row varchar(10));
CREATE TABLE
test=# CREATE SUBSCRIPTION subscr_test1
CONNECTION 'host=192.168.186.173 port=5432 user=postgres password=postgres dbname=test'
PUBLICATION public_test1 WITH (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "subscr_test1"
CREATE SUBSCRIPTION
test=# \dRs
                  Список подписок
     Имя      | Владелец | Включён |   Публикация
--------------+----------+---------+----------------
 subscr_test1 | postgres | t       | {public_test1}
(1 строка)

test=# SELECT i, row FROM test2;
 i |    row
---+-----------
 1 | Значение3
 2 | Значение4
 3 | Значение5
(3 строки)

test=# SELECT i, row FROM test1;
 i |    row
---+-----------
 1 | Значение1
 2 | Значение2
 3 | Значение3
(3 строки)

test=# SELECT * FROM pg_stat_subscription \gx
-[ RECORD 1 ]---------+------------------------------
subid                 | 16399
subname               | subscr_test1
pid                   | 1638
relid                 |
received_lsn          | 0/19B59A0
last_msg_send_time    | 2023-07-23 13:32:04.311242+03
last_msg_receipt_time | 2023-07-23 13:32:04.312407+03
latest_end_lsn        | 0/19B59A0
latest_end_time       | 2023-07-23 13:32:04.311242+03
```
_Подписка на публикацию таблицы test1 с первой ВМ выполнена._

_Настраиваем третью ВМ alse174cmdt5 IP 192.168.186.175 как реплику для чтения и бэкапов. Подписываемся на таблицы из первой alse174cmdt3 IP 192.168.186.173 и второй ВМ alse174cmdt4 IP 192.168.186.174._

## 3
```
postgres@alse174cmdt5:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# ALTER SYSTEM SET wal_level = logical;
ALTER SYSTEM
postgres=# CREATE DATABASE test;
CREATE DATABASE
postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=# CREATE TABLE test1 (i int, row varchar(10));
CREATE TABLE
test=# CREATE TABLE test2 (i int, row varchar(10));
CREATE TABLE
test=# \q
postgres@alse174cmdt5:~$ pg_ctl stop
ожидание завершения работы сервера.... готово
сервер остановлен
```

__Создаем подписку на третьей ВМ alse174cmdt4 с первой ВМ alse174cmdt3 IP 192.168.186.173 и со второй ВМ alse174cmdt4 IP 192.168.186.174. Проверяем состояние подписки. Делаем запосы к таблицам.__

## 3
```
postgres@alse174cmdt5:~$ pg_ctl start
ожидание запуска сервера....2023-07-23 13:42:08.053 MSK [1165] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
2023-07-23 13:42:08.053 MSK [1165] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
 готово
сервер запущен
postgres@alse174cmdt5:~$ pg_ctl status
pg_ctl: сервер работает (PID: 1165)
/opt/tantor/db/15/bin/postgres
postgres@alse174cmdt5:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=# CREATE SUBSCRIPTION subscr_test1_vm3
test-# CONNECTION 'host=192.168.186.173 port=5432 user=postgres password=postgres dbname=test'
test-# PUBLICATION public_test1 WITH (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "subscr_test1_vm3"
CREATE SUBSCRIPTION
test=# CREATE SUBSCRIPTION subscr_test2_vm3
test-# CONNECTION 'host=192.168.186.174 port=5432 user=postgres password=postgres dbname=test'
test-# PUBLICATION public_test2 WITH (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "subscr_test2_vm3"
CREATE SUBSCRIPTION
test=# \dRs
                    Список подписок
       Имя        | Владелец | Включён |   Публикация
------------------+----------+---------+----------------
 subscr_test1_vm3 | postgres | t       | {public_test1}
 subscr_test2_vm3 | postgres | t       | {public_test2}
(2 строки)

test=# SELECT * FROM pg_stat_subscription \gx
-[ RECORD 1 ]---------+------------------------------
subid                 | 16397
subname               | subscr_test1_vm3
pid                   | 1335
relid                 |
received_lsn          | 0/19B5B58
last_msg_send_time    | 2023-07-23 13:47:27.112406+03
last_msg_receipt_time | 2023-07-23 13:47:27.113126+03
latest_end_lsn        | 0/19B5B58
latest_end_time       | 2023-07-23 13:47:27.112406+03
-[ RECORD 2 ]---------+------------------------------
subid                 | 16398
subname               | subscr_test2_vm3
pid                   | 1338
relid                 |
received_lsn          | 0/19CA288
last_msg_send_time    | 2023-07-23 13:47:13.092881+03
last_msg_receipt_time | 2023-07-23 13:47:13.09288+03
latest_end_lsn        | 0/19CA288
latest_end_time       | 2023-07-23 13:47:13.092881+03

test=# SELECT i, row FROM test1;
 i |    row
---+-----------
 1 | Значение1
 2 | Значение2
 3 | Значение3
(3 строки)

test=# SELECT i, row FROM test2;
 i |    row
---+-----------
 1 | Значение3
 2 | Значение4
 3 | Значение5
(3 строки)

```
_Подписка на публикацию таблиц test1 и test2 выполнена._

_Проверяем состояние репликации и информацию о слоьах репликации на первой ВМ._

## 1
```
test=# SELECT * FROM pg_stat_replication \gx
-[ RECORD 1 ]----+------------------------------
pid              | 1554
usesysid         | 10
usename          | postgres
application_name | subscr_test1
client_addr      | 192.168.186.174
client_hostname  |
client_port      | 50968
backend_start    | 2023-07-23 13:31:04.152767+03
backend_xmin     |
state            | streaming
sent_lsn         | 0/19B5B58
write_lsn        | 0/19B5B58
flush_lsn        | 0/19B5B58
replay_lsn       | 0/19B5B58
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2023-07-23 13:48:37.224455+03
-[ RECORD 2 ]----+------------------------------
pid              | 1619
usesysid         | 10
usename          | postgres
application_name | subscr_test1_vm3
client_addr      | 192.168.186.175
client_hostname  |
client_port      | 38534
backend_start    | 2023-07-23 13:46:57.009878+03
backend_xmin     |
state            | streaming
sent_lsn         | 0/19B5B58
write_lsn        | 0/19B5B58
flush_lsn        | 0/19B5B58
replay_lsn       | 0/19B5B58
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2023-07-23 13:48:37.233002+03

test=# SELECT * FROM pg_replication_slots \gx
-[ RECORD 1 ]-------+-----------------
slot_name           | subscr_test1
plugin              | pgoutput
slot_type           | logical
datoid              | 16388
database            | test
temporary           | f
active              | t
active_pid          | 1554
xmin                |
catalog_xmin        | 751
restart_lsn         | 0/19B5B10
confirmed_flush_lsn | 0/19B5B58
wal_status          | reserved
safe_wal_size       |
two_phase           | f
-[ RECORD 2 ]-------+-----------------
slot_name           | subscr_test1_vm3
plugin              | pgoutput
slot_type           | logical
datoid              | 16388
database            | test
temporary           | f
active              | t
active_pid          | 1619
xmin                |
catalog_xmin        | 751
restart_lsn         | 0/19B5B10
confirmed_flush_lsn | 0/19B5B58
wal_status          | reserved
safe_wal_size       |
two_phase           | f

test=#
```

_Проверяем состояние репликации и информацию о слоьах репликации на второй ВМ._

## 2
```
test=# SELECT * FROM pg_stat_replication \gx
-[ RECORD 1 ]----+------------------------------
pid              | 1620
usesysid         | 10
usename          | postgres
application_name | subscr_test2
client_addr      | 192.168.186.173
client_hostname  |
client_port      | 50448
backend_start    | 2023-07-23 13:26:47.167615+03
backend_xmin     |
state            | streaming
sent_lsn         | 0/19CA288
write_lsn        | 0/19CA288
flush_lsn        | 0/19CA288
replay_lsn       | 0/19CA288
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2023-07-23 13:49:23.301378+03
-[ RECORD 2 ]----+------------------------------
pid              | 1697
usesysid         | 10
usename          | postgres
application_name | subscr_test2_vm3
client_addr      | 192.168.186.175
client_hostname  |
client_port      | 57712
backend_start    | 2023-07-23 13:47:13.047124+03
backend_xmin     |
state            | streaming
sent_lsn         | 0/19CA288
write_lsn        | 0/19CA288
flush_lsn        | 0/19CA288
replay_lsn       | 0/19CA288
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2023-07-23 13:49:23.311862+03

test=# SELECT * FROM pg_replication_slots \gx
-[ RECORD 1 ]-------+-----------------
slot_name           | subscr_test2
plugin              | pgoutput
slot_type           | logical
datoid              | 16388
database            | test
temporary           | f
active              | t
active_pid          | 1620
xmin                |
catalog_xmin        | 753
restart_lsn         | 0/19CA240
confirmed_flush_lsn | 0/19CA288
wal_status          | reserved
safe_wal_size       |
two_phase           | f
-[ RECORD 2 ]-------+-----------------
slot_name           | subscr_test2_vm3
plugin              | pgoutput
slot_type           | logical
datoid              | 16388
database            | test
temporary           | f
active              | t
active_pid          | 1697
xmin                |
catalog_xmin        | 753
restart_lsn         | 0/19CA240
confirmed_flush_lsn | 0/19CA288
wal_status          | reserved
safe_wal_size       |
two_phase           | f
```

_Настраиваем горячее реплицирование для высокой доступности на четвертой ВМ alse174cmdt3 IP 192.168.186.176. Источник репликации третья ВМ alse174cmdt5 IP 192.168.186.175._

_Проверяем, и, при необходимости, выполняем настройки на третьей ВМ alse174cmdt5 IP 192.168.186.175. Перезагружаем инстанс._

## 3
```
test=# SELECT current_setting('synchronous_commit');
 current_setting
-----------------
 on
(1 строка)

test=# SELECT current_setting('max_wal_senders');
 current_setting
-----------------
 10
(1 строка)

test=# SELECT current_setting('hot_standby');
 current_setting
-----------------
 on
(1 строка)

test=# ALTER SYSTEM SET synchronous_standby_names = '*';
ALTER SYSTEM
test=# SELECT  pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 строка)

test=# \q
postgres@alse174cmdt5:~$ pg_ctl stop
ожидание завершения работы сервера.... готово
сервер остановлен
postgres@alse174cmdt5:~$ pg_ctl start
ожидание запуска сервера....2023-07-23 14:08:56.446 MSK [1512] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
2023-07-23 14:08:56.446 MSK [1512] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
 готово
сервер запущен
postgres@alse174cmdt5:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=#
```

_Проверяем, и, при необходимости, выполняем настройки на четвертой ВМ alse174cmdt6 IP 192.168.186.176. Перезагружаем инстанс. Удаляем каталог с данными БД на четвертой ВМ. Копируем каталог с данными БД с третьей ВМ alse174cmdt5 IP 192.168.186.175 программой pg_basebackup. Ключи программы pg_basebackup P показывает прогресс выполнения, R создает базовый вариант файла управления recovery.conf с записью конфигурацию для репликации, Х включает в копию требуемые файлы WAL, используя заданный метод._

## 4
```
test=# SELECT current_setting('synchronous_commit');
 current_setting
-----------------
 on
(1 строка)

test=# SELECT current_setting('max_wal_senders');
 current_setting
-----------------
 10
(1 строка)

test=# SELECT current_setting('hot_standby');
 current_setting
-----------------
 on
(1 строка)

test=# ALTER SYSTEM SET synchronous_standby_names = '*';
ALTER SYSTEM
test=# SELECT  pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 строка)

test=# \q
postgres@alse174cmdt6:~$ pg_ctl stop
ожидание завершения работы сервера.... готово
сервер остановлен
postgres@alse174cmdt6:~$ cd /var/lib/postgresql/tantor-se-15/
postgres@alse174cmdt6:~/tantor-se-15$ ls -lha
итого 12K
drwxr-xr-x  3 postgres postgres 4,0K июл  5 21:47 .
drwx------  4 postgres postgres 4,0K июл 23 13:57 ..
drwx------ 20 postgres postgres 4,0K июл 23 14:00 data
postgres@alse174cmdt6:~/tantor-se-15$ rm -r data/
postgres@alse174cmdt6:~/tantor-se-15$ ls -lha
итого 8,0K
drwxr-xr-x 2 postgres postgres 4,0K июл 23 14:00 .
drwx------ 4 postgres postgres 4,0K июл 23 13:57 ..
postgres@alse174cmdt6:~/tantor-se-15$ mkdir data
postgres@alse174cmdt6:~/tantor-se-15$ chmod go-rwx data
postgres@alse174cmdt6:~/tantor-se-15$ ls -lha
итого 12K
drwxr-xr-x 3 postgres postgres 4,0K июл 23 14:00 .
drwx------ 4 postgres postgres 4,0K июл 23 13:57 ..
drwx------ 2 postgres postgres 4,0K июл 23 14:00 data
postgres@alse174cmdt6:~/tantor-se-15$ pg_basebackup -P -R -X stream -h 192.168.186.175 -U postgres -D ./data
Пароль:
30614/30614 КБ (100%), табличное пространство 1/1
postgres@alse174cmdt6:~/tantor-se-15$ ls -lha data/
итого 320K
drwx------ 20 postgres postgres 4,0K июл 23 14:11 .
drwxr-xr-x  3 postgres postgres 4,0K июл 23 14:10 ..
-rw-------  1 postgres postgres  225 июл 23 14:11 backup_label
-rw-------  1 postgres postgres 179K июл 23 14:11 backup_manifest
drwx------  6 postgres postgres 4,0K июл 23 14:11 base
-rw-------  1 postgres postgres   44 июл 23 14:11 current_logfiles
drwx------  2 postgres postgres 4,0K июл 23 14:11 global
drwx------  2 postgres postgres 4,0K июл 23 14:11 log
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_commit_ts
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_dynshmem
-rw-------  1 postgres postgres 4,9K июл 23 14:11 pg_hba.conf
-rw-------  1 postgres postgres 1,6K июл 23 14:11 pg_ident.conf
drwx------  4 postgres postgres 4,0K июл 23 14:11 pg_logical
drwx------  4 postgres postgres 4,0K июл 23 14:11 pg_multixact
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_notify
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_replslot
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_serial
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_snapshots
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_stat
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_stat_tmp
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_subtrans
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_tblspc
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_twophase
-rw-------  1 postgres postgres    3 июл 23 14:11 PG_VERSION
drwx------  3 postgres postgres 4,0K июл 23 14:11 pg_wal
drwx------  2 postgres postgres 4,0K июл 23 14:11 pg_xact
-rw-------  1 postgres postgres  402 июл 23 14:11 postgresql.auto.conf
-rw-------  1 postgres postgres  30K июл 23 14:11 postgresql.conf
-rw-------  1 postgres postgres    0 июл 23 14:11 standby.signal
postgres@alse174cmdt6:~$ pg_ctl start
ожидание запуска сервера....2023-07-23 14:13:13.880 MSK [1180] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
2023-07-23 14:13:13.880 MSK [1180] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
 готово
сервер запущен
postgres@alse174cmdt6:~$ pg_ctl status
pg_ctl: сервер работает (PID: 1180)
/opt/tantor/db/15/bin/postgres
```
_Дополним таблицу test2 на второй ВМ._
## 2
```
test=# INSERT INTO test2 VALUES(4, 'Значение6');
INSERT 0 1
test=# INSERT INTO test2 VALUES(5, 'Значение7');
INSERT 0 1
test=# INSERT INTO test2 VALUES(6, 'Значение8');
INSERT 0 1

test=# SELECT i, row FROM test2;
 i |    row
---+-----------
 1 | Значение3
 2 | Значение4
 3 | Значение5
 4 | Значение6
 5 | Значение7
 6 | Значение8
 
(6 строк)

test=#
```

_Выполним запрос данных таблицы test2 на четвертой ВМ._

## 4
```
postgres@alse174cmdt6:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# \c test
Вы подключены к базе данных "test" как пользователь "postgres".
test=# \dt
          Список отношений
 Схема  |  Имя  |   Тип   | Владелец
--------+-------+---------+----------
 public | test1 | таблица | postgres
 public | test2 | таблица | postgres
(2 строки)

test=# SELECT i, row FROM test2;
 i |    row
---+-----------
 1 | Значение3
 2 | Значение4
 3 | Значение5
 4 | Значение6
 5 | Значение7
 6 | Значение8
 
(6 строк)

test=#
```
_Проверим состояние репликации на третьей ВМ. _
## 3
```
test=# SELECT * FROM pg_stat_replication \gx
-[ RECORD 1 ]----+------------------------------
pid              | 1542
usesysid         | 10
usename          | postgres
application_name | walreceiver
client_addr      | 192.168.186.176
client_hostname  |
client_port      | 38796
backend_start    | 2023-07-23 14:13:13.923399+03
backend_xmin     |
state            | streaming
sent_lsn         | 0/5000918
write_lsn        | 0/5000918
flush_lsn        | 0/5000918
replay_lsn       | 0/5000918
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 1
sync_state       | sync
reply_time       | 2023-07-23 14:19:09.895664+03

test=# SELECT pg_current_wal_lsn();
 pg_current_wal_lsn
--------------------
 0/5000918
(1 строка)

test=#
```

_Поверяем состояние репликации на четвертой ВМ. Уточняем время крайнего обновления данных._

## 4
```
test=# SELECT pg_last_wal_receive_lsn();
 pg_last_wal_receive_lsn
-------------------------
 0/5000918
(1 строка)

test=# SELECT  pg_last_wal_replay_lsn();
 pg_last_wal_replay_lsn
------------------------
 0/5000918
(1 строка)

test=# SELECT * from pg_stat_wal_receiver \gx
-[ RECORD 1 ]---------+-------------------------------------------------------------------------------------------------------------------------------------                                     ------------------------------------------------------------------------------------------------------------------------------------------------------------                                     -------
pid                   | 1185
status                | streaming
receive_start_lsn     | 0/5000000
receive_start_tli     | 1
written_lsn           | 0/5000918
flushed_lsn           | 0/5000918
received_tli          | 1
last_msg_send_time    | 2023-07-23 14:20:39.944871+03
last_msg_receipt_time | 2023-07-23 14:20:39.945037+03
latest_end_lsn        | 0/5000918
latest_end_time       | 2023-07-23 14:17:39.835135+03
slot_name             |
sender_host           | 192.168.186.175
sender_port           | 5432
conninfo              | user=postgres password=******** channel_binding=prefer dbname=replication host=192.168.186.175 port=5432 fallback_application_name=w                                     alreceiver sslmode=prefer sslcompression=0 sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres compression=off target_session_at                                     trs=any

test=# SELECT * from pg_stat_wal_receiver \gx
-[ RECORD 1 ]---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
pid                   | 1185
status                | streaming
receive_start_lsn     | 0/5000000
receive_start_tli     | 1
written_lsn           | 0/5000918
flushed_lsn           | 0/5000918
received_tli          | 1
last_msg_send_time    | 2023-07-23 14:20:39.944871+03
last_msg_receipt_time | 2023-07-23 14:20:39.945037+03
latest_end_lsn        | 0/5000918
latest_end_time       | 2023-07-23 14:17:39.835135+03
slot_name             |
sender_host           | 192.168.186.175
sender_port           | 5432
conninfo              | user=postgres password=******** channel_binding=prefer dbname=replication host=192.168.186.175 port=5432 fallback_application_name=walreceiver sslmode=prefer sslcompression=0 sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres compression=off target_session_attrs=any

test=# SELECT now()-pg_last_xact_replay_timestamp();
    ?column?
-----------------
 00:03:48.301521
(1 строка)

test=#
```