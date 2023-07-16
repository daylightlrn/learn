# Домашнее задание к лекции "Настройка PostgreSQL"
## Дата: 13.07.2023
## Тема: Настройка PostgreSQL
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ (4 core, 16 Gb, 20 Gb), ОС AstraLinux 1.7.4 mode 0, ПО Тантор (СУБД 15)

## Описание/Пошаговая инструкция выполнения домашнего задания:
_• развернуть виртуальную машину любым удобным способом
• поставить на неё PostgreSQL 15 любым способом
• настроить кластер PostgreSQL 15 на максимальную производительность не
обращая внимание на возможные проблемы с надежностью в случае
аварийной перезагрузки виртуальной машины
• нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
• написать какого значения tps удалось достичь, показать какие параметры в
какие значения устанавливали и почему
Задание со *: аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench)._


## Выполнение
_Подготовка. PgBench входит в состав дистрибутива Tantor. Устанавливаем SysBench. Поскольку скрипт установки SysBench проверяет ОС, а ОС ALSE в списке нет, необходимо установить превентивно имя ОС и тип дистрибутива, соответствующие ALSE 1.7.x.(os=debian dist=buster)._

## 1
```
sudo curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh > script.sh
chmod u+x script.sh
sudo os=debian dist=buster ./script.sh
Detected operating system as debian/buster.
Checking for curl...
...
The repository is setup! You can now install packages.
sudo apt -y install sysbench
Чтение списков пакетов… Готово
Построение дерева зависимостей
Чтение информации о состоянии… Готово
...
```


_Проведем тесты на СУБД в два этапа, на первом с настройками по умолчанию, а затем, на втором, изменим конфигурацию СУБД. Размеры тестовых БД сделаем одинаковыми 2,1 Гб. (в PgBench настраивается параметром scale, для SysBench количеством строк в таблице oltp-table-size и общим числом таблиц oltp-tables-count). Время выполнения теста в обоих случаях 10 мин., количество подключений 80._

## 1
```

```
_Выполняем тесты с использованием PgBench (конфигурация СУБД по умолчанию). Описание используемых PgBench ключей приведено ниже для создания тестовой БД (pgbench -i -s 141 <ИМЯ_ТЕСТОВОЙ_БД> -h <IP_Адрес_СУБД> -p <Порт_СУБД> -U <ИМЯ_ПОЛЬЗОВАТЕЛЯ_СУБД>).
-i Инициализировать БД
-s Коэффициент умножения для генерируемых строк БД, позволяет создавть БД заданного размера._

## 1
```
postgres@alse174cmdt3:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# CREATE DATABASE pgbenchtst;
CREATE DATABASE
postgres=# CREATE DATABASE sysbenchtst;
CREATE DATABASE
postgres=# ALTER USER POSTGRES WITH ENCRYPTED PASSWORD 'postgres';
ALTER ROLE
postgres=# \l+
                                                                                         Список баз данных
     Имя     | Владелец | Кодировка | LC_COLLATE  |  LC_CTYPE   | локаль ICU | Провайдер локали |     Права доступа     | Размер  | Табл. пространство |                  Описание
-------------+----------+-----------+-------------+-------------+------------+------------------+-----------------------+---------+--------------------+--------------------------------------------
 pgbenchtst  | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             |                       | 7361 kB | pg_default         |
 postgres    | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             |                       | 7441 kB | pg_default         | default administrative connection database
 sysbenchtst | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             |                       | 7361 kB | pg_default         |
 template0   | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             | =c/postgres          +| 7281 kB | pg_default         | unmodifiable empty database
             |          |           |             |             |            |                  | postgres=CTc/postgres |         |                    |
 template1   | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             | =c/postgres          +| 7521 kB | pg_default         | default template for new databases
             |          |           |             |             |            |                  | postgres=CTc/postgres |         |                    |
(5 строк)
postgres=# \q
postgres@alse174cmdt3:~$ pgbench -i -s 141 pgbenchtst -h 192.168.186.173 -p 5432 -U postgres
Password:
dropping old tables...
ЗАМЕЧАНИЕ:  таблица "pgbench_accounts" не существует, пропускается
ЗАМЕЧАНИЕ:  таблица "pgbench_branches" не существует, пропускается
ЗАМЕЧАНИЕ:  таблица "pgbench_history" не существует, пропускается
ЗАМЕЧАНИЕ:  таблица "pgbench_tellers" не существует, пропускается
creating tables...
generating data (client-side)...
14100000 of 14100000 tuples (100%) done (elapsed 33.59 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 40.98 s (drop tables 0.00 s, create tables 0.02 s, client-side generate 34.28 s, vacuum 0.32 s, primary keys 6.36 s).
```

_Убедимся, что размер тестовой БД pgbenchtst 2.1 Гб._ 
## 1
```
postgres@alse174cmdt3:~$ psql -c '\c pgbenchtst' -c '\dt++'
Вы подключены к базе данных "pgbenchtst" как пользователь "postgres".
                                         Список отношений
 Схема  |       Имя        |   Тип   | Владелец |  Хранение  | Метод доступа | Размер  | Описание
--------+------------------+---------+----------+------------+---------------+---------+----------
 public | pgbench_accounts | таблица | postgres | постоянное | heap          | 1834 MB |
 public | pgbench_branches | таблица | postgres | постоянное | heap          | 456 kB  |
 public | pgbench_history  | таблица | postgres | постоянное | heap          | 29 MB   |
 public | pgbench_tellers  | таблица | postgres | постоянное | heap          | 448 kB  |
(4 строки)
```

_Запускаем тест.
pgbench -c80 -P 60 -T 600 -h <IP_Адрес_СУБД> -p <Порт_СУБД> -U <ИМЯ_ПОЛЬЗОВАТЕЛЯ_СУБД> <ИМЯ_ТЕСТОВОЙ_БД>
-c Количество подключающихся клиентов.
-P Показывать отчет о проделанной работе каждые 60 секунд.
-T Запустите тест на это количество секунд._

## 1
```
postgres@alse174cmdt3:~$ pgbench -c80 -P 60 -T 600 -h 192.168.186.173 -p 5432 -U postgres pgbenchtst
Password:
pgbench (15.2)
starting vacuum...end.
progress: 60.0 s, 940.5 tps, lat 83.693 ms stddev 47.731, 0 failed
progress: 120.0 s, 950.2 tps, lat 84.160 ms stddev 53.921, 0 failed
progress: 180.0 s, 926.2 tps, lat 86.355 ms stddev 49.588, 0 failed
progress: 240.0 s, 928.8 tps, lat 86.121 ms stddev 55.756, 0 failed
progress: 300.0 s, 932.1 tps, lat 85.766 ms stddev 55.418, 0 failed
progress: 360.0 s, 968.5 tps, lat 82.624 ms stddev 49.625, 0 failed
progress: 420.0 s, 950.8 tps, lat 84.108 ms stddev 51.333, 0 failed
progress: 480.0 s, 968.5 tps, lat 82.578 ms stddev 50.395, 0 failed
progress: 540.0 s, 962.0 tps, lat 83.152 ms stddev 51.726, 0 failed
progress: 600.0 s, 963.2 tps, lat 83.027 ms stddev 50.445, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 141
query mode: simple
number of clients: 80
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 569534
number of failed transactions: 0 (0.000%)
latency average = 84.142 ms
latency stddev = 51.657 ms
initial connection time = 905.245 ms
tps = 950.377876 (without initial connection time)
postgres@alse174cmdt3:~$

```
_Наполняем тестовую БД SysBench. Выполняем тесты с использованием SysBench (конфигурация СУБД по умолчанию). Используем аналогичные параметры, как и для PgBench. В отличии от параметра -s PgBench, определяющего размер всей БД, в случае SysBench задаем количество строк в таблице (--oltp-table-size=1000000) и кол-во таблиц (--oltp-tables-count=10). И, таким образом, формируем необходимый размер БД. 2.1 Гб._

## 1
```
sudo sysbench \
 --db-driver=pgsql \
 --oltp-table-size=1000000 \
 --oltp-tables-count=10 \
 --threads=1 \
 --pgsql-host=192.168.186.173 \
 --pgsql-port=5432 \
 --pgsql-user=postgres \
 --pgsql-password=postgres \
 --pgsql-db=sysbenchtst \
 /usr/share/sysbench/tests/include/oltp_legacy/parallel_prepare.lua \
run
sysbench 1.0.18 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Initializing worker threads...

Threads started!

thread prepare0
Creating table 'sbtest1'...
Inserting 1000000 records into 'sbtest1'
Creating secondary indexes on 'sbtest1'...
...
Creating table 'sbtest10'...
Inserting 1000000 records into 'sbtest10'
Creating secondary indexes on 'sbtest10'...
SQL statistics:
    queries performed:
        read:                            0
        write:                           3730
        other:                           20
        total:                           3750
    transactions:                        1      (0.01 per sec.)
    queries:                             3750   (32.12 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          116.7328s
    total number of events:              1

Latency (ms):
         min:                               116732.35
         avg:                               116732.35
         max:                               116732.35
         95th percentile:                   100000.00
         sum:                               116732.35

Threads fairness:
    events (avg/stddev):           1.0000/0.00
    execution time (avg/stddev):   116.7324/0.00

postgres@alse174cmdt3:~$ psql -c '\c sysbenchtst' -c '\dt++'
Вы подключены к базе данных "sysbenchtst" как пользователь "postgres".
                                    Список отношений
 Схема  |   Имя    |   Тип   | Владелец |  Хранение  | Метод доступа | Размер | Описание
--------+----------+---------+----------+------------+---------------+--------+----------
 public | sbtest1  | таблица | postgres | постоянное | heap          | 211 MB |
 public | sbtest10 | таблица | postgres | постоянное | heap          | 211 MB |
 public | sbtest2  | таблица | postgres | постоянное | heap          | 211 MB |
 public | sbtest3  | таблица | postgres | постоянное | heap          | 211 MB |
 public | sbtest4  | таблица | postgres | постоянное | heap          | 211 MB |
 public | sbtest5  | таблица | postgres | постоянное | heap          | 211 MB |
 public | sbtest6  | таблица | postgres | постоянное | heap          | 211 MB |
 public | sbtest7  | таблица | postgres | постоянное | heap          | 211 MB |
 public | sbtest8  | таблица | postgres | постоянное | heap          | 211 MB |
 public | sbtest9  | таблица | postgres | постоянное | heap          | 211 MB |
(10 строк)
```

_Запускаем тест SysBench._

## 1
```
postgres@alse174cmdt3:~$ sysbench \
 --db-driver=pgsql \
 --report-interval=60 \
 --oltp-table-size=1000000 \
 --oltp-tables-count=10 \
 --threads=80 \
 --time=600 \
 --pgsql-host=192.168.186.173 \
 --pgsql-port=5432 \
 --pgsql-user=postgres \
 --pgsql-password=postgres \
 --pgsql-db=sysbenchtst \
 /usr/share/sysbench/tests/include/oltp_legacy/oltp.lua \
run
sysbench 1.0.18 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 80
Report intermediate results every 60 second(s)
Initializing random number generator from current time


Initializing worker threads...

Threads started!

[ 60s ] thds: 80 tps: 222.09 qps: 4455.10 (r/w/o: 3120.26/889.36/445.49) lat (ms,95%): 549.52 err/s: 0.02 reconn/s: 0.00
[ 120s ] thds: 80 tps: 231.19 qps: 4629.21 (r/w/o: 3241.44/925.35/462.42) lat (ms,95%): 539.71 err/s: 0.00 reconn/s: 0.00
[ 180s ] thds: 80 tps: 227.63 qps: 4547.40 (r/w/o: 3182.30/909.69/455.41) lat (ms,95%): 539.71 err/s: 0.03 reconn/s: 0.00
[ 240s ] thds: 80 tps: 220.93 qps: 4422.44 (r/w/o: 3096.05/884.50/441.89) lat (ms,95%): 569.67 err/s: 0.00 reconn/s: 0.00
[ 300s ] thds: 80 tps: 226.16 qps: 4522.48 (r/w/o: 3166.27/903.86/452.35) lat (ms,95%): 559.50 err/s: 0.00 reconn/s: 0.00
[ 360s ] thds: 80 tps: 223.44 qps: 4471.04 (r/w/o: 3130.18/893.88/446.97) lat (ms,95%): 580.02 err/s: 0.02 reconn/s: 0.00
[ 420s ] thds: 80 tps: 224.38 qps: 4489.51 (r/w/o: 3142.76/897.83/448.91) lat (ms,95%): 569.67 err/s: 0.02 reconn/s: 0.00
[ 480s ] thds: 80 tps: 225.57 qps: 4511.74 (r/w/o: 3158.72/901.78/451.24) lat (ms,95%): 539.71 err/s: 0.02 reconn/s: 0.00
[ 540s ] thds: 80 tps: 225.03 qps: 4497.14 (r/w/o: 3147.18/899.88/450.08) lat (ms,95%): 539.71 err/s: 0.02 reconn/s: 0.00
[ 600s ] thds: 80 tps: 214.97 qps: 4301.53 (r/w/o: 3010.50/861.03/430.00) lat (ms,95%): 559.50 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            1884106
        write:                           538279
        other:                           269181
        total:                           2691566
    transactions:                        134572 (224.11 per sec.)
    queries:                             2691566 (4482.47 per sec.)
    ignored errors:                      7      (0.01 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          600.4635s
    total number of events:              134572

Latency (ms):
         min:                                    3.66
         avg:                                  356.77
         max:                                 1940.19
         95th percentile:                      559.50
         sum:                             48011757.99

Threads fairness:
    events (avg/stddev):           1682.1500/20.64
    execution time (avg/stddev):   600.1470/0.16

postgres@alse174cmdt3:~$
```

_Изменяем конфигурацию СУБД в конфигурационном файле postgresql.conf. Перезапускаем СУБД._

## 1
```
shared_buffers = '4892 MB'             # Рекомендуется устанавливать 1/4 от общего объема памяти
work_mem = '50 MB'                     # Может принимать значения 2-3% от общего объема памяти, в нашем случае объем памяти позволяет установить 50 MB
maintenance_work_mem = '100 MB'
effective_cache_size = '7 GB'          # Рекомендуется устанавливать 2/3 от общего объема памяти
effective_io_concurrency = 100         # concurrent IO only really activated if OS supports posix_fadvise function

checkpoint_timeout  = '1 h'            # Увеличиваем время записи данных на диск при контрольной точке, тем самым уменьшаем количество обращений к диску
max_wal_size = '2048 MB'               # Так же увеличиваем размер файла журнала

synchronous_commit = off               # Увеличиваем скорость записи, теряем надежность 
fsync = off                            # Увеличиваем скорость записи, теряем надежность
full_page_writes = off                 # Устанавливаем в **off**, в случае fsync **off**

max_worker_processes = 4               # Соответствует количеству ядер процессора 
max_parallel_workers_per_gather = 2    # Количество параллельных процессов, задействованных в запросе, выделяется из max_parallel_workers
max_parallel_maintenance_workers = 2   # количество параллельных процессов на операцию обслуживания, выделяется из max_parallel_workers
max_parallel_workers = 4               # Количество max_worker_processes, используемых в параллельных процессах, меньше или равно max_worker_processes
```

_Выполняем тесты с использованием PgBench (конфигурация СУБД изменена)._

## 1
```
postgres@alse174cmdt3:~$ pgbench -c80 -P 60 -T 600 -h 192.168.186.173 -p 5432 -U postgres pgbenchtst
Password:
pgbench (15.2)
starting vacuum...end.
progress: 60.0 s, 2027.0 tps, lat 38.638 ms stddev 13.100, 0 failed
progress: 120.0 s, 2218.8 tps, lat 35.969 ms stddev 12.429, 0 failed
progress: 180.0 s, 2241.0 tps, lat 35.567 ms stddev 12.297, 0 failed
progress: 240.0 s, 2271.1 tps, lat 35.095 ms stddev 11.630, 0 failed
progress: 300.0 s, 2280.0 tps, lat 34.965 ms stddev 11.565, 0 failed
progress: 360.0 s, 2298.2 tps, lat 34.669 ms stddev 10.406, 0 failed
progress: 420.0 s, 2308.6 tps, lat 34.518 ms stddev 11.865, 0 failed
progress: 480.0 s, 2319.7 tps, lat 34.336 ms stddev 10.478, 0 failed
progress: 540.0 s, 2289.0 tps, lat 34.800 ms stddev 12.191, 0 failed
progress: 600.0 s, 2308.4 tps, lat 34.509 ms stddev 11.506, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 141
query mode: simple
number of clients: 80
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 1353794
number of failed transactions: 0 (0.000%)
latency average = 35.265 ms
latency stddev = 11.815 ms
initial connection time = 1110.085 ms
tps = 2259.985255 (without initial connection time)
postgres@alse174cmdt3:~$
```
_Выполняем тесты с использованием SysBench (конфигурация СУБД изменена)._

## 1
```
postgres@alse174cmdt3: sysbench \
 --db-driver=pgsql \
 --report-interval=60 \
 --oltp-table-size=1000000 \
 --oltp-tables-count=10 \
 --threads=80 \
 --time=600 \
 --pgsql-host=192.168.186.173 \
 --pgsql-port=5432 \
 --pgsql-user=postgres \
 --pgsql-password=postgres \
 --pgsql-db=sysbenchtst \
 /usr/share/sysbench/tests/include/oltp_legacy/oltp.lua \
run
sysbench 1.0.18 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 80
Report intermediate results every 60 second(s)
Initializing random number generator from current time


Initializing worker threads...

Threads started!

[ 60s ] thds: 80 tps: 398.27 qps: 7980.41 (r/w/o: 5589.14/1593.36/797.91) lat (ms,95%): 337.94 err/s: 0.00 reconn/s: 0.00
[ 120s ] thds: 80 tps: 437.08 qps: 8742.19 (r/w/o: 6119.52/1748.52/874.15) lat (ms,95%): 244.38 err/s: 0.00 reconn/s: 0.00
[ 180s ] thds: 80 tps: 442.07 qps: 8839.40 (r/w/o: 6186.77/1768.40/884.23) lat (ms,95%): 240.02 err/s: 0.02 reconn/s: 0.00
[ 240s ] thds: 80 tps: 441.83 qps: 8839.82 (r/w/o: 6188.64/1767.39/883.79) lat (ms,95%): 235.74 err/s: 0.05 reconn/s: 0.00
[ 300s ] thds: 80 tps: 442.35 qps: 8845.68 (r/w/o: 6191.75/1769.14/884.79) lat (ms,95%): 240.02 err/s: 0.02 reconn/s: 0.00
[ 360s ] thds: 80 tps: 421.93 qps: 8440.96 (r/w/o: 5909.11/1687.81/844.03) lat (ms,95%): 257.95 err/s: 0.05 reconn/s: 0.00
[ 420s ] thds: 80 tps: 427.10 qps: 8542.41 (r/w/o: 5979.53/1708.52/854.35) lat (ms,95%): 257.95 err/s: 0.03 reconn/s: 0.00
[ 480s ] thds: 80 tps: 424.26 qps: 8485.36 (r/w/o: 5939.55/1697.16/848.65) lat (ms,95%): 253.35 err/s: 0.03 reconn/s: 0.00
[ 540s ] thds: 80 tps: 414.69 qps: 8293.86 (r/w/o: 5805.90/1658.33/829.63) lat (ms,95%): 272.27 err/s: 0.05 reconn/s: 0.00
[ 600s ] thds: 80 tps: 410.75 qps: 8216.03 (r/w/o: 5751.29/1643.18/821.56) lat (ms,95%): 272.27 err/s: 0.02 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            3580038
        write:                           1022800
        other:                           511470
        total:                           5114308
    transactions:                        255701 (425.97 per sec.)
    queries:                             5114308 (8519.83 per sec.)
    ignored errors:                      16     (0.03 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          600.2808s
    total number of events:              255701

Latency (ms):
         min:                                    3.21
         avg:                                  187.75
         max:                                 1889.11
         95th percentile:                      257.95
         sum:                             48006925.78

Threads fairness:
    events (avg/stddev):           3196.2625/28.34
    execution time (avg/stddev):   600.0866/0.10

postgres@alse174cmdt3:~$
```

_В результате получили значительный прирост в количестве транзакций и скорости записи, но потеряли надежность._