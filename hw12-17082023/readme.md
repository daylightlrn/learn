# Домашнее задание к лекции "Секционирование"
## Дата: 17.08.2023
## Тема: Секционирование таблицы
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ alse174cmdt3, IP 192.168.186.173, экран  1, (2 core, 4 Gb, 20 Gb), ОС AstraLinux 1.7.4 mode 0, ПО Тантор (СУБД 15)
## Описание/Пошаговая инструкция выполнения домашнего задания:
_Секционировать большую таблицу из демо базы flights_

## Выполнение

_Загружаем, разархивируем и импортируем демонстрационную БД_

## 1
```
postgres@alse174cmdt3:~$ wget https://edu.postgrespro.ru/demo-big.zip
--2023-09-02 18:19:52--  https://edu.postgrespro.ru/demo-big.zip
Распознаётся edu.postgrespro.ru (edu.postgrespro.ru)… 213.171.56.196
Подключение к edu.postgrespro.ru (edu.postgrespro.ru)|213.171.56.196|:443... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 243203214 (232M) [application/zip]
Сохранение в: «demo-big.zip»

demo-big.zip                            100%[==============================================================================>] 231,94M  14,1MB/s    за 17s

2023-09-02 18:20:08 (14,0 MB/s) - «demo-big.zip» сохранён [243203214/243203214]

postgres@alse174cmdt3:~$ ls -lha

-rw-r--r--  1 postgres postgres 232M янв 11  2018 demo-big.zip

postgres@alse174cmdt3:~$ mkdir bigdataset
postgres@alse174cmdt3:~$ mv demo-big.zip bigdataset/

postgres@alse174cmdt3:~$ ls -lha bigdataset/
-rw-r--r-- 1 postgres postgres 232M янв 11  2018 demo-big.zip

postgres@alse174cmdt3:~$ cd bigdataset/
postgres@alse174cmdt3:~/bigdataset$ unzip demo-big.zip -d ./
Archive:  demo-big.zip
  inflating: ./demo-big-20170815.sql
postgres@alse174cmdt3:~/bigdataset$ ls -lha
-rw-rw-r-- 1 postgres postgres 888M янв  5  2018 demo-big-20170815.sql
-rw-r--r-- 1 postgres postgres 232M янв 11  2018 demo-big.zip

postgres@alse174cmdt3:~/bigdataset$ psql -f ./demo-big-20170815.sql
SET
...
ALTER DATABASE
```

_Проверяем БД, подключаемся к БД, смотрим список таблиц_

## 1
```
postgres=# \l
                                                  Список баз данных
    Имя    | Владелец | Кодировка | LC_COLLATE  |  LC_CTYPE   | локаль ICU | Провайдер локали |     Права доступа
-----------+----------+-----------+-------------+-------------+------------+------------------+-----------------------
 demo      | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             |
 postgres  | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             |
 template0 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             | =c/postgres          +
           |          |           |             |             |            |                  | postgres=CTc/postgres
 template1 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             | =c/postgres          +
           |          |           |             |             |            |                  | postgres=CTc/postgres
(4 строки)

postgres=# \c demo
Вы подключены к базе данных "demo" как пользователь "postgres".
demo=# \dt+
                                                 Список отношений
  Схема   |       Имя       |   Тип   | Владелец |  Хранение  | Метод доступа | Размер |         Описание
----------+-----------------+---------+----------+------------+---------------+--------+---------------------------
 bookings | aircrafts_data  | таблица | postgres | постоянное | heap          | 16 kB  | Aircrafts (internal data)
 bookings | airports_data   | таблица | postgres | постоянное | heap          | 56 kB  | Airports (internal data)
 bookings | boarding_passes | таблица | postgres | постоянное | heap          | 459 MB | Boarding passes
 bookings | bookings        | таблица | postgres | постоянное | heap          | 106 MB | Bookings
 bookings | flights         | таблица | postgres | постоянное | heap          | 21 MB  | Flights
 bookings | seats           | таблица | postgres | постоянное | heap          | 96 kB  | Seats
 bookings | ticket_flights  | таблица | postgres | постоянное | heap          | 551 MB | Flight segment
 bookings | tickets         | таблица | postgres | постоянное | heap          | 387 MB | Tickets
(8 строк)
```

_Для секционирования выбираем таблицу bookings, смотрим информацию об этой таблице_

## 1
```
demo=# \d bookings
                                   Таблица "bookings.bookings"
   Столбец    |           Тип            | Правило сортировки | Допустимость NULL | По умолчанию
--------------+--------------------------+--------------------+-------------------+--------------
 book_ref     | character(6)             |                    | not null          |
 book_date    | timestamp with time zone |                    | not null          |
 total_amount | numeric(10,2)            |                    | not null          |
Индексы:
    "bookings_pkey" PRIMARY KEY, btree (book_ref)
Ссылки извне:
    TABLE "tickets" CONSTRAINT "tickets_book_ref_fkey" FOREIGN KEY (book_ref) REFERENCES bookings(book_ref)
```

_Определяем граничные значения для секций_

## 1
```
demo=# select min(book_date),max(book_date) from bookings;
          min           |          max
------------------------+------------------------
 2016-07-20 21:16:00+03 | 2017-08-15 18:00:00+03
(1 строка)
```

_Определяем время испольнения запроса до секционирования_

## 1
```
demo=# EXPLAIN ANALYZE
demo-# SELECT * FROM bookings.bookings WHERE book_date BETWEEN date'2016-11-01' AND date'2017-03-01'-1;
                                                    QUERY PLAN
-------------------------------------------------------------------------------------------------------------------
 Seq Scan on bookings  (cost=0.00..45199.65 rows=654103 width=21) (actual time=0.022..399.746 rows=656980 loops=1)
   Filter: ((book_date >= '2016-11-01'::date) AND (book_date <= '2017-02-28'::date))
   Rows Removed by Filter: 1454130
 Planning Time: 0.140 ms
 Execution Time: 421.486 ms
(5 строк)
```

_Создаем объект секционирования_

## 1
```
demo=# CREATE TABLE bookings.the_part_of_bookings (
    book_ref character(6) NOT NULL,
    book_date timestamp with time zone NOT NULL,
    total_amount numeric(10,2) NOT NULL
) PARTITION BY RANGE(book_date);
```

_Разбиваем таблицы по месяцам_

## 1
```
demo=# CREATE TABLE bookings.the_part_of_bookings_2016_07 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2016-07-01') TO ('2016-08-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2016_08 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2016-08-01') TO ('2016-09-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2016_09 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2016-09-01') TO ('2016-10-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2016_10 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2016-10-01') TO ('2016-11-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2016_11 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2016-11-01') TO ('2016-12-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2016_12 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2016-12-01') TO ('2017-01-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2017_01 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2017-01-01') TO ('2017-02-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2017_02 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2017-02-01') TO ('2017-03-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2017_03 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2017-03-01') TO ('2017-04-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2017_04 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2017-04-01') TO ('2017-05-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2017_05 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2017-05-01') TO ('2017-06-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2017_06 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2017-06-01') TO ('2017-07-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2017_07 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2017-07-01') TO ('2017-08-01');
CREATE TABLE
demo=# CREATE TABLE bookings.the_part_of_bookings_2017_08 PARTITION OF bookings.the_part_of_bookings FOR VALUES FROM ('2017-08-01') TO ('2017-09-01');
CREATE TABLE
```

_Копируем данные во вновь созданную таблицу the_part_of_bookings

## 1
```
demo=# INSERT INTO bookings.the_part_of_bookings (book_ref,book_date,total_amount) SELECT book_ref,book_date,total_amount FROM bookings.bookings;
INSERT 0 2111110
```

_Повторно выполняем запрос_

## 1
```
demo=# EXPLAIN ANALYZE
SELECT * FROM bookings.the_part_of_bookings WHERE book_date BETWEEN date'2016-11-01' AND date'2017-03-01'-1;
                                                                            QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Append  (cost=0.00..48494.68 rows=657606 width=21) (actual time=0.043..200.504 rows=656980 loops=1)
   Subplans Removed: 10
   ->  Seq Scan on the_part_of_bookings_2016_11 the_part_of_bookings_1  (cost=0.00..3543.03 rows=165436 width=21) (actual time=0.042..54.586 rows=165469 loops=1)
         Filter: ((book_date >= '2016-11-01'::date) AND (book_date <= '2017-02-28'::date))
   ->  Seq Scan on the_part_of_bookings_2016_12 the_part_of_bookings_2  (cost=0.00..3666.97 rows=171231 width=21) (actual time=0.021..37.783 rows=171265 loops=1)
         Filter: ((book_date >= '2016-11-01'::date) AND (book_date <= '2017-02-28'::date))
   ->  Seq Scan on the_part_of_bookings_2017_01 the_part_of_bookings_3  (cost=0.00..3665.74 rows=171149 width=21) (actual time=0.015..32.632 rows=171183 loops=1)
         Filter: ((book_date >= '2016-11-01'::date) AND (book_date <= '2017-02-28'::date))
   ->  Seq Scan on the_part_of_bookings_2017_02 the_part_of_bookings_4  (cost=0.00..3309.68 rows=149780 width=21) (actual time=0.024..28.956 rows=149063 loops=1)
         Filter: ((book_date >= '2016-11-01'::date) AND (book_date <= '2017-02-28'::date))
         Rows Removed by Filter: 5516
 Planning Time: 3.971 ms
 Execution Time: 224.923 ms
(13 строк)

```

_Время уменьшилось, поскольку обращение происходит к необходимым секциям_