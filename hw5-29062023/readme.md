# Домашнее задание к лекции "MVCC, vacuum и autovacuum"
## Дата: 29.06.2023
## Тема: MVCC в PostgreSQL Vacuum & Autovacuum
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ (2 core, 4 Gb, 20 Gb), ОС AstraLinux 1.7.4 mode 0, ПО Тантор (СУБД 15)

## Описание/Пошаговая инструкция выполнения домашнего задания:

_оздать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
Установить на него PostgreSQL 15 с дефолтными настройками
Создать БД для тестов: выполнить pgbench -i postgres
Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
Протестировать заново
Что изменилось и почему?
Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
Посмотреть размер файла с таблицей
5 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
Подождать некоторое время, проверяя, пришел ли автовакуум
5 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей
Отключить Автовакуум на конкретной таблице
10 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей
Объясните полученный результат
Не забудьте включить автовакуум)
Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла._

## Выполнение
_Создаем виртуальную машину с указанными параметрами._

## 1
```
root@alse174cmdt3:~# nproc
2
root@alse174cmdt3:~# grep MemTotal /proc/meminfo
MemTotal:        3985452 kB
root@alse174cmdt3:~#
```
_Устанавливаем СУБД. Настраиваем автозапуск. Проверяем сетевой доступ._

## 1
```
root@alse174cmdt3:~# wget --quiet -O - https://public.tantorlabs.ru/tantorlabs.ru.asc | apt-key add -
OK
root@alse174cmdt3:~# echo "deb [arch=amd64] https://<USER_LOGIN>:<USER_PASSWORD>@nexus.tantorlabs.ru/repository/astra-smolensk-1.7 smolensk main" > /etc/apt/sources.list.d/tantorlabs.list
root@alse174cmdt3:~# apt-get update
Сущ:1 https://download.astralinux.ru/astra/stable/1.7_x86-64/repository-base 1.7_x86-64 InRelease
Пол:2 https://nexus.tantorlabs.ru/repository/astra-smolensk-1.7 smolensk InRelease [1 554 B]
Сущ:3 https://download.astralinux.ru/astra/stable/1.7_x86-64/repository-extended 1.7_x86-64 InRelease
...
root@alse174cmdt3:~# su - postgres
postgres@alse174cmdt3:~$ initdb -D /var/lib/postgresql/tantor-se-15/data --no-instructions 2> /dev/null
Файлы, относящиеся к этой СУБД, будут принадлежать пользователю "postgres".
От его имени также будет запускаться процесс сервера.

Кластер баз данных будет инициализирован с локалью "ru_RU.UTF-8".
...
postgres@alse174cmdt3:~$ выход
root@alse174cmdt3:~# systemctl status tantor-se-server-15.service
● tantor-se-server-15.service - Tantor Special Edition database server 15
   Loaded: loaded (/lib/systemd/system/tantor-se-server-15.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: https://tantorlabs.ru/docs/
root@alse174cmdt3:~# systemctl enable tantor-se-server-15.service
Created symlink /etc/systemd/system/multi-user.target.wants/tantor-se-server-15.service → /lib/systemd/system/tantor-se-server-15.service.
root@alse174cmdt3:~# systemctl status tantor-se-server-15.service
● tantor-se-server-15.service - Tantor Special Edition database server 15
   Loaded: loaded (/lib/systemd/system/tantor-se-server-15.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2023-07-05 21:57:49 MSK; 1min 11s ago
...
root@alse174cmdt3:~# nmap -sT -p 5432 192.168.186.173
Starting Nmap 7.70 ( https://nmap.org ) at 2023-07-05 22:06 MSK
Nmap scan report for alse174cmdt3.example.dn (192.168.186.173)
Host is up (0.00023s latency).

PORT     STATE SERVICE
5432/tcp open  postgresql

Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
root@alse174cmdt3:~#
```
_Создаем БД для тестов._
```
postgres@alse174cmdt3:~$ pgbench -i postgres
dropping old tables...
ЗАМЕЧАНИЕ:  таблица "pgbench_accounts" не существует, пропускается
ЗАМЕЧАНИЕ:  таблица "pgbench_branches" не существует, пропускается
ЗАМЕЧАНИЕ:  таблица "pgbench_history" не существует, пропускается
ЗАМЕЧАНИЕ:  таблица "pgbench_tellers" не существует, пропускается
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.20 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.62 s (drop tables 0.00 s, create tables 0.01 s, client-side generate 0.25 s, vacuum 0.17 s, primary keys 0.19 s).
postgres@alse174cmdt3:~$
```
_Выполним тестирование СУБД._
```
postgres@alse174cmdt3:~$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.2)
starting vacuum...end.
progress: 6.0 s, 1230.2 tps, lat 6.383 ms stddev 2.950, 0 failed
progress: 12.0 s, 1218.2 tps, lat 6.564 ms stddev 3.225, 0 failed
progress: 18.0 s, 1202.3 tps, lat 6.652 ms stddev 3.254, 0 failed
progress: 24.0 s, 1193.3 tps, lat 6.704 ms stddev 4.340, 0 failed
progress: 30.0 s, 1209.8 tps, lat 6.609 ms stddev 3.134, 0 failed
progress: 36.0 s, 1197.3 tps, lat 6.681 ms stddev 3.264, 0 failed
progress: 42.0 s, 1229.0 tps, lat 6.508 ms stddev 3.210, 0 failed
progress: 48.0 s, 1209.3 tps, lat 6.612 ms stddev 4.970, 0 failed
progress: 54.0 s, 1250.5 tps, lat 6.398 ms stddev 3.079, 0 failed
progress: 60.0 s, 1206.3 tps, lat 6.630 ms stddev 3.232, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 72886
number of failed transactions: 0 (0.000%)
latency average = 6.573 ms
latency stddev = 3.519 ms
initial connection time = 107.027 ms
tps = 1216.709772 (without initial connection time)
postgres@alse174cmdt3:~$
```
_Применим новые параметры._
```
postgres@alse174cmdt3:~$ psql -c "alter system set max_connections = 40;"
ALTER SYSTEM
postgres@alse174cmdt3:~$ psql -c "alter system set shared_buffers = '1GB';"
ALTER SYSTEM
postgres@alse174cmdt3:~$ psql -c "alter system set effective_cache_size = '3GB';"
ALTER SYSTEM
postgres@alse174cmdt3:~$ psql -c "alter system set maintenance_work_mem = '512MB';"
ALTER SYSTEM
postgres@alse174cmdt3:~$ psql -c "alter system set checkpoint_completion_target = 0.9;"
ALTER SYSTEM
postgres@alse174cmdt3:~$ psql -c "alter system set wal_buffers = '16MB';"
ALTER SYSTEM
postgres@alse174cmdt3:~$ psql -c "alter system set default_statistics_target = 500;"
ALTER SYSTEM
postgres@alse174cmdt3:~$ psql -c "alter system set random_page_cost = 4;"
ALTER SYSTEM
postgres@alse174cmdt3:~$ psql -c "alter system set effective_io_concurrency = 2;"
ALTER SYSTEM
postgres@alse174cmdt3:~$ psql -c "alter system set work_mem = '6553kB';"
ALTER SYSTEM
postgres@alse174cmdt3:~$ psql -c "alter system set min_wal_size = '4GB';"
ALTER SYSTEM
postgres@alse174cmdt3:~$ psql -c "alter system set max_wal_size = '16GB';"
ALTER SYSTEM
postgres@alse174cmdt3:~$
```
_После установки параметров необходимо перезапустить СУБД._
```
postgres@alse174cmdt3:~$ pg_ctl stop
ожидание завершения работы сервера.... готово
сервер остановлен
postgres@alse174cmdt3:~$ pg_ctl start
ожидание запуска сервера....2023-07-05 22:30:50.080 MSK [1550] СООБЩЕНИЕ:  запускается PostgreSQL 15.2 on x86_64-pc-linux-gnu,                                             compiled by gcc (AstraLinuxSE 8.3.0-6) 8.3.0, 64-bit
2023-07-05 22:30:50.081 MSK [1550] СООБЩЕНИЕ:  для приёма подключений по адресу IPv4 "0.0.0.0" открыт порт 5432
2023-07-05 22:30:50.081 MSK [1550] СООБЩЕНИЕ:  для приёма подключений по адресу IPv6 "::" открыт порт 5432
2023-07-05 22:30:50.083 MSK [1550] СООБЩЕНИЕ:  для приёма подключений открыт Unix-сокет "/var/run/postgresql/.s.PGSQL.5432"
2023-07-05 22:30:50.090 MSK [1553] СООБЩЕНИЕ:  система БД была выключена: 2023-07-05 22:30:45 MSK
2023-07-05 22:30:50.095 MSK [1550] СООБЩЕНИЕ:  система БД готова принимать подключения
 готово
сервер запущен
postgres@alse174cmdt3:~$
```
_Измененные параметры попадают в файл postgresql.auto.conf. Проверим значения примененных параметров._
```
postgres@alse174cmdt3:~$ psql -c "select * from pg_file_settings where sourcefile='/var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf' order by name;"
                         sourcefile                         | sourceline | seqno |             name             | setting | applied | error
------------------------------------------------------------+------------+-------+------------------------------+---------+---------+-------
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |          7 |    19 | checkpoint_completion_target | 0.9     | t       |
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |          9 |    21 | default_statistics_target    | 500     | t       |
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |          5 |    17 | effective_cache_size         | 3GB     | t       |
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |         11 |    23 | effective_io_concurrency     | 2       | t       |
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |          6 |    18 | maintenance_work_mem         | 512MB   | t       |
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |          3 |    15 | max_connections              | 40      | t       |
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |         14 |    26 | max_wal_size                 | 16GB    | t       |
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |         13 |    25 | min_wal_size                 | 4GB     | t       |
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |         10 |    22 | random_page_cost             | 4       | t       |
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |          4 |    16 | shared_buffers               | 1GB     | t       |
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |          8 |    20 | wal_buffers                  | 16MB    | t       |
 /var/lib/postgresql/tantor-se-15/data/postgresql.auto.conf |         12 |    24 | work_mem                     | 6553kB  | t       |
(12 строк)

postgres@alse174cmdt3:~$
```
_Выполним тетстировани СУБД с новыми параметрами._
```
postgres@alse174cmdt3:~$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.2)
starting vacuum...end.
progress: 6.0 s, 1146.0 tps, lat 6.803 ms stddev 3.858, 0 failed
progress: 12.0 s, 1223.7 tps, lat 6.455 ms stddev 3.363, 0 failed
progress: 18.0 s, 1209.2 tps, lat 6.523 ms stddev 3.429, 0 failed
progress: 24.0 s, 1227.5 tps, lat 6.433 ms stddev 3.263, 0 failed
progress: 30.0 s, 1248.5 tps, lat 6.326 ms stddev 3.286, 0 failed
progress: 36.0 s, 1287.7 tps, lat 6.129 ms stddev 2.829, 0 failed
progress: 42.0 s, 1253.9 tps, lat 6.294 ms stddev 3.052, 0 failed
progress: 48.0 s, 1244.9 tps, lat 6.345 ms stddev 3.341, 0 failed
progress: 54.0 s, 1224.3 tps, lat 6.448 ms stddev 3.386, 0 failed
progress: 60.0 s, 1243.3 tps, lat 6.345 ms stddev 3.142, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 73864
number of failed transactions: 0 (0.000%)
latency average = 6.406 ms
latency stddev = 3.303 ms
initial connection time = 75.787 ms
tps = 1232.243953 (without initial connection time)
postgres@alse174cmdt3:~$
```
_Результат сравнения двух тестов показывает небольшое ускорение работы СУБД. Увеличение количества транзакций в секунду (tps) 1250.5->1287.7. Увеличение количества обработанных транзакций (number of transactions actually processed) 72886->73864. Уменьшение средней латентности (latency average) 6.573->6.406. Уменьшение заддержки(латентности) ввода/вывода (latency stddev) 3.519->3.303. Уменьшение времени подключения (initial connection time)107.027->75.787.



_Создаем таблицу с 1 млн. строк. Получаем размер таблицы. Обновляем 5 раз все строки._
```
postgres@alse174cmdt3:~$ psql -c "INSERT INTO testtable1(c1) SELECT 'noname' FROM generate_series(1,1000000);"
INSERT 0 1000000
postgres@alse174cmdt3:~$ psql -c "SELECT pg_size_pretty(pg_table_size('testtable1'));"
 pg_size_pretty
----------------
 35 MB
(1 строка)

postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'1';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'2';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'3';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'4';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'5';"
UPDATE 1000000
postgres@alse174cmdt3:~$
```
_Поскольку autovacuum уже прошел (2023-07-05 23:15), количество строк мы не увидим. Ожидаемое количество строк 5 млн._
```
postgres@alse174cmdt3:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'testtable1';
  relname   | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
------------+------------+------------+--------+-------------------------------
 testtable1 |    1000000 |          0 |      0 | 2023-07-05 23:15:52.167269+03
(1 строка)

postgres=# \q
postgres@alse174cmdt3:~$
```
_Обновляем 5 раз все строки._

```
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'6';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'7';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'8';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'9';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'10';"
UPDATE 1000000
postgres@alse174cmdt3:~$
```
_Получаем размер таблицы._
```
psql -c "SELECT pg_size_pretty(pg_table_size('testtable1'));"
 pg_size_pretty
----------------
 261 MB
(1 строка)

postgres@alse174cmdt3:~$
```
_Отключаем autovacuum._
```
postgres@alse174cmdt3:~$ psql -c "alter table testtable1 set (autovacuum_enabled = off);"
ALTER TABLE
postgres@alse174cmdt3:~$
```
_Обновляем 10 раз все строки._

```
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'11';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'12';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'13';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'14';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'15';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'16';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'17';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'18';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'19';"
UPDATE 1000000
postgres@alse174cmdt3:~$ psql -c "UPDATE testtable1 set c1=c1||'20';"
UPDATE 1000000
postgres@alse174cmdt3:~$
```
_Получаем размер таблицы._
```
postgres@alse174cmdt3:~$ psql -c "SELECT pg_size_pretty(pg_table_size('testtable1'));"
 pg_size_pretty
----------------
 629 MB
(1 строка)
postgres@alse174cmdt3:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'testtable1';
  relname   | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
------------+------------+------------+--------+-------------------------------
 testtable1 |    1000000 |    9995881 |    999 | 2023-07-05 23:36:53.405231+03
(1 строка)

postgres=# \q
postgres@alse174cmdt3:~$
```
_Autovacuum выключен, строки остались в количестве ~ 10 млн. Для оптимизации размера таблицы необходимо выполнить полный autovacuum._