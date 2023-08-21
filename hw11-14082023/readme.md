# Домашнее задание к лекции "Сбор и использование статистики"
## Дата: 14.08.2023
## Тема: Репликация
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ alse174cmdt3, IP 192.168.186.173, экран  1, (2 core, 4 Gb, 20 Gb), ОС AstraLinux 1.7.4 mode 0, ПО Тантор (СУБД 15)
## Описание/Пошаговая инструкция выполнения домашнего задания:
_1 вариант:

Создать индексы на БД, которые ускорят доступ к данным.

В данном задании тренируются навыки:

определения узких мест

написания запросов для создания индекса оптимизации

Необходимо:

Создать индекс к какой-либо из таблиц вашей БД

Прислать текстом результат команды explain, в которой используется данный индекс

Реализовать индекс для полнотекстового поиска

Реализовать индекс на часть таблицы или индекс на поле с функцией

Создать индекс на несколько полей

Написать комментарии к каждому из индексов

Описать что и как делали и с какими проблемами столкнулись_

_2 вариант:

В результате выполнения ДЗ вы научитесь пользоваться различными вариантами соединения таблиц.

В данном задании тренируются навыки:

написания запросов с различными типами соединений

Необходимо:

Реализовать прямое соединение двух или более таблиц

Реализовать левостороннее (или правостороннее) соединение двух или более таблиц

Реализовать кросс соединение двух или более таблиц

Реализовать полное соединение двух или более таблиц

Реализовать запрос, в котором будут использованы разные типы соединений

Сделать комментарии на каждый запрос

К работе приложить структуру таблиц, для которых выполнялись соединения_

_Задание со звездочкой*

Придумайте 3 своих метрики на основе показанных представлений, отправьте их через ЛК, а так же поделитесь с коллегами в слак_


## Выполнение

_Оценка выполнения времени запроса производится с использованием команды psql \timing_

_Для подавления вывода при замерах времени выполнения запроса применяется перенаправление в "нулевое устройство" \o /dev/null_


_ВАРИАНТ 1_

_Выполняем подготовительные работы по созданию и наполнению таблицы._

## 1
```
postgres=# CREATE TABLE purchases(id int, buyer_id int, purchase_date date, status text, some_txt text);
CREATE TABLE
postgres=# INSERT INTO purchases(id, buyer_id, purchase_date, status, some_txt)
postgres-# SELECT generate_series, (random() * 70), date'2023-01-01' + (random() * 300)::int AS order_date
postgres-#         , (array['returned', 'completed', 'placed', 'shipped'])[(random() * 4)::int]
postgres-#         , concat_ws(' ', (array['go', 'space', 'sun', 'Moscow'])[(random() * 5)::int]
postgres(#         , (array['the', 'capital', 'of', 'Russian', 'Federation'])[(random() * 6)::int]
postgres(#         , (array['some', 'another', 'example', 'with', 'words'])[(random() * 6)::int]
postgres(#         )
postgres-# FROM generate_series(1, 5000000);
INSERT 0 5000000
postgres=#
```

_Делаем оценку плана запроса_


## 1
```
postgres=# EXPLAIN
postgres-# SELECT * FROM purchases WHERE id<1000000;
                                   QUERY PLAN
---------------------------------------------------------------------------------
 Seq Scan on purchases  (cost=0.00..103171.30 rows=1017439 width=35)
   Filter: (id < 1000000)
 JIT:
   Functions: 2
   Options: Inlining false, Optimization false, Expressions true, Deforming true
(5 строк)

postgres=#
```
_и выполняем запрос_

## 1
```
postgres=# \o /dev/null
postgres=# \timing
Секундомер включён.
postgres=# SELECT * FROM purchases WHERE id<1000000;
Время: 1471,184 мс (00:01,471)
postgres=# \o
postgres=# \timing
Секундомер выключен.
postgres=#
```
_Создаем индекс на одно поле по id_

## 1
```
postgres=# CREATE UNIQUE INDEX idx_ord_id ON purchases(id);
CREATE INDEX
postgres=#
```

_и проверяем будет ли он использоваться_


## 1
```
postgres=# EXPLAIN
postgres-# SELECT * FROM purchases WHERE id<1000000;
                                      QUERY PLAN
---------------------------------------------------------------------------------------
 Index Scan using idx_ord_id on purchases  (cost=0.43..37249.53 rows=1017434 width=35)
   Index Cond: (id < 1000000)
(2 строки)

postgres=#
```

_По результатам анализа видно, что используется Index Scan_


_Убедимся, что время выполнения запроса уменьшилось_

## 1
```
postgres=# \o /dev/null
postgres=# \timing
Секундомер включён.
postgres=# SELECT * FROM purchases WHERE id<1000000;
Время: 913,152 мс
postgres=# \o
postgres=# \timing
Секундомер выключен.
postgres=#
```

_Реализовать индекс для полнотекстового поиска_

_Добавим столбец с типом tsvector и заполним его данными из столбца some_txt_

## 1
```
postgres=# ALTER TABLE purchases ADD COLUMN some_txt_lexeme tsvector;
ALTER TABLE
postgres=# UPDATE purchases SET some_txt_lexeme = to_tsvector(some_txt);
UPDATE 5000000
postgres=#
```
_Выполним анализ плана запроса_

## 1
```
postgres=# EXPLAIN
postgres-# SELECT some_txt_lexeme
postgres-# FROM purchases
postgres-# WHERE some_txt_lexeme @@ to_tsquery('federations');
                                    QUERY PLAN
-----------------------------------------------------------------------------------
 Gather  (cost=1000.00..1439809.26 rows=60962 width=32)
   Workers Planned: 2
   ->  Parallel Seq Scan on purchases  (cost=0.00..1432713.06 rows=25401 width=32)
         Filter: (some_txt_lexeme @@ to_tsquery('federations'::text))
 JIT:
   Functions: 4
   Options: Inlining true, Optimization true, Expressions true, Deforming true
(7 строк)

postgres=#
```
_Используется Parallel Seq Scan, выполняем запрос_

## 1
```
postgres=# \o /dev/null
postgres=# \timing
Секундомер включён.
postgres=# SELECT some_txt
FROM purchases
WHERE some_txt_lexeme @@ to_tsquery('federations');
Время: 12683,833 мс (00:12,684)
postgres=# \o
postgres=# \timing
Секундомер выключен.
postgres=#
```
_Оптимизируем время и добавляем индекс с методом доступа GIN_

## 1
```
postgres=# CREATE INDEX search_index_pur ON purchases USING GIN (some_txt_lexeme);
CREATE INDEX
postgres=#
```
_Выполним проверку плана запроса_

## 1
```
postgres-# WHERE some_txt_lexeme @@ to_tsquery('federations');
                                          QUERY PLAN
----------------------------------------------------------------------------------------------
 Gather  (cost=8763.17..588705.40 rows=841667 width=29)
   Workers Planned: 2
   ->  Parallel Bitmap Heap Scan on purchases  (cost=7763.17..503538.70 rows=350695 width=29)
         Recheck Cond: (some_txt_lexeme @@ to_tsquery('federations'::text))
         ->  Bitmap Index Scan on search_index_pur  (cost=0.00..7552.75 rows=841667 width=0)
               Index Cond: (some_txt_lexeme @@ to_tsquery('federations'::text))
 JIT:
   Functions: 4
   Options: Inlining true, Optimization true, Expressions true, Deforming true
(9 строк)

postgres=#
```
_и сам запрос_

## 1
```
postgres=# \o /dev/null
postgres=# \timing
Секундомер включён.
postgres=# SELECT some_txt_lexeme
postgres-# FROM purchases
postgres-# WHERE some_txt_lexeme @@ to_tsquery('federations');
Время: 1953,293 мс (00:01,953)
postgres=# \o
postgres=# \timing
Секундомер выключен.
postgres=#
```
_Время запроса уменьшилось_

_Индекс на часть таблицы или индекс на поле с функцией_

## 1
```
postgres=# CREATE UNIQUE INDEX idx_pur_id30 ON purchases(id) WHERE id<30;
CREATE INDEX
postgres=#
```
_План запроса_

## 1
```
postgres=# EXPLAIN
postgres-# SELECT * FROM purchases WHERE id<30;
                                  QUERY PLAN
------------------------------------------------------------------------------
 Index Scan using idx_ord_id on purchases  (cost=0.43..9.54 rows=29 width=64)
   Index Cond: (id < 30)
(2 строки)

postgres=#
```
_Тип алгоритма Index Scan_

_Индекс на несколько полей_

_План запроса_

## 1
```
postgres=# EXPLAIN
postgres-# SELECT purchase_date, status
postgres-# FROM purchases
postgres-# WHERE purchase_date BETWEEN date'2023-07-01' AND date'2023-08-01'
postgres-#         AND STATUS = 'placed';
                                                           QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..149484.33 rows=128510 width=12)
   Workers Planned: 2
   ->  Parallel Seq Scan on purchases  (cost=0.00..135633.33 rows=53546 width=12)
         Filter: ((purchase_date >= '2023-07-01'::date) AND (purchase_date <= '2023-08-01'::date) AND (status = 'placed'::text))
 JIT:
   Functions: 4
   Options: Inlining false, Optimization false, Expressions true, Deforming true
(7 строк)

postgres=#
```
_Алгоритм Parallel Seq Scan_

## 1
```
postgres=# \o /dev/null
postgres=# \timing
Секундомер включён.
postgres=# SELECT purchase_date, status
postgres-# FROM purchases
postgres-# WHERE purchase_date BETWEEN date'2023-07-01' AND date'2023-08-01'
postgres-#          AND STATUS = 'placed';
Время: 1012,220 мс (00:01,012)
postgres=# \o
postgres=# \timing
Секундомер выключен.
postgres=#
```
_Создаем индекс на несколько полей_

## 1
```
postgres=# CREATE INDEX idx_pur_purchase_date_status ON purchases(purchase_date, status);
CREATE INDEX
postgres=#
```
_План запроса_

## 1
```
postgres=# EXPLAIN
postgres-# SELECT purchase_date, status
postgres-# FROM purchases
postgres-# WHERE purchase_date BETWEEN date'2023-07-01' AND date'2023-08-01'
postgres-#         AND STATUS = 'placed';
                                                          QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------------
 Index Only Scan using idx_pur_purchase_date_status on purchases  (cost=0.43..9466.17 rows=128510 width=12)
   Index Cond: ((purchase_date >= '2023-07-01'::date) AND (purchase_date <= '2023-08-01'::date) AND (status = 'placed'::text))
(2 строки)

postgres=#
```
_Используется Index Only Scan_

_Выполняем запрос_

## 1
```
postgres=# \o /dev/null
postgres=# \timing
Секундомер включён.
postgres=# SELECT purchase_date, status
postgres-# FROM purchases
postgres-# WHERE purchase_date BETWEEN date'2023-07-01' AND date'2023-08-01'
postgres-#          AND STATUS = 'placed';
Время: 59,865 мс
postgres=# \o
postgres=# \timing
Секундомер выключен.
postgres=#
```
_Время выполнения запроса уменьшилось_

_ВАРИАНТ 2_

_Для выполнения соединения таблиц нам необходима еще одна таблица, содаем ее_

## 1
```
postgres=# CREATE TABLE customers(buyer_id int, passwd text);
CREATE TABLE
postgres=# INSERT INTO customers(buyer_id, passwd)
postgres-# SELECT generate_series, md5(random()::text) FROM generate_series(1, 100);
INSERT 0 100
postgres=#
```
_Создадим индексы для полей buyer_id для каждой таблицы_

## 1
```
postgres=# CREATE INDEX idx_cust_uid on customers(buyer_id);
CREATE INDEX
postgres=# CREATE INDEX idx_pur_uid on purchases(buyer_id);
CREATE INDEX
postgres=#
```
_Прямое соединение_

_Выполнение плана запроса при соединении двух таблиц с условием customers.buyer_id = purchases.buyer_id_

## 1
```
postgres=# EXPLAIN ANALYZE
postgres-# SELECT a.buyer_id, b.purchase_date FROM customers A INNER JOIN purchases b on a.buyer_id=b.buyer_id;
                                                          QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------
 Hash Join  (cost=3.25..217928.25 rows=5000000 width=8) (actual time=9.394..2289.411 rows=4964220 loops=1)
   Hash Cond: (b.buyer_id = a.buyer_id)
   ->  Seq Scan on purchases b  (cost=0.00..149175.00 rows=5000000 width=8) (actual time=0.078..903.470 rows=5000000 loops=1)
   ->  Hash  (cost=2.00..2.00 rows=100 width=4) (actual time=9.301..9.303 rows=100 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 12kB
         ->  Seq Scan on customers a  (cost=0.00..2.00 rows=100 width=4) (actual time=9.245..9.275 rows=100 loops=1)
 Planning Time: 1.296 ms
 JIT:
   Functions: 11
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 1.871 ms, Inlining 0.000 ms, Optimization 0.676 ms, Emission 8.366 ms, Total 10.913 ms
 Execution Time: 2498.255 ms
(12 строк)

postgres=#
```
_Используется Hash Join_

_Левостороннее соединение_

## 1
```
postgres=# EXPLAIN ANALYZE
postgres-# SELECT a.buyer_id, b.buyer_id FROM customers A LEFT JOIN purchases b ON a.buyer_id=b.buyer_id;
                                                                      QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------------------------------
 Merge Left Join  (cost=771.68..154450.25 rows=5000000 width=8) (actual time=12.227..1397.051 rows=4964250 loops=1)
   Merge Cond: (a.buyer_id = b.buyer_id)
   ->  Sort  (cost=5.32..5.57 rows=100 width=4) (actual time=4.336..4.428 rows=100 loops=1)
         Sort Key: a.buyer_id
         Sort Method: quicksort  Memory: 25kB
         ->  Seq Scan on customers a  (cost=0.00..2.00 rows=100 width=4) (actual time=4.307..4.324 rows=100 loops=1)
   ->  Index Only Scan using idx_pur_uid on purchases b  (cost=0.43..91944.43 rows=5000000 width=4) (actual time=0.118..674.976 rows=5000000 loops=1)
         Heap Fetches: 0
 Planning Time: 0.184 ms
 JIT:
   Functions: 7
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 0.915 ms, Inlining 0.000 ms, Optimization 0.275 ms, Emission 3.843 ms, Total 5.033 ms
 Execution Time: 1611.544 ms
(14 строк)

postgres=#
```

_Используется Merge Left Join_

_Кросс соединение_

_План запроса_


## 1
```
postgres=# EXPLAIN
postgres-# SELECT a.buyer_id, b.buyer_id FROM customers A CROSS JOIN purchases b;
                                             QUERY PLAN
----------------------------------------------------------------------------------------------------
 Nested Loop  (cost=0.43..6341946.68 rows=500000000 width=8)
   ->  Index Only Scan using idx_pur_uid on purchases b  (cost=0.43..91944.43 rows=5000000 width=4)
   ->  Materialize  (cost=0.00..2.50 rows=100 width=4)
         ->  Seq Scan on customers a  (cost=0.00..2.00 rows=100 width=4)
 JIT:
   Functions: 4
   Options: Inlining true, Optimization true, Expressions true, Deforming true
(7 строк)

postgres=#
```

_Выполняем запрос с анализом_

## 1
```
postgres=# EXPLAIN ANALYZE
postgres-# SELECT a.buyer_id, b.buyer_id FROM customers A CROSS JOIN purchases b;
                                                                      QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------------------------------------
 Nested Loop  (cost=0.43..6341946.68 rows=500000000 width=8) (actual time=34.087..79860.985 rows=500000000 loops=1)
   ->  Index Only Scan using idx_pur_uid on purchases b  (cost=0.43..91944.43 rows=5000000 width=4) (actual time=0.035..1085.453 rows=5000000 loops=1)
         Heap Fetches: 0
   ->  Materialize  (cost=0.00..2.50 rows=100 width=4) (actual time=0.000..0.006 rows=100 loops=5000000)
         ->  Seq Scan on customers a  (cost=0.00..2.00 rows=100 width=4) (actual time=34.042..34.053 rows=100 loops=1)
 Planning Time: 0.086 ms
 JIT:
   Functions: 4
   Options: Inlining true, Optimization true, Expressions true, Deforming true
   Timing: Generation 0.558 ms, Inlining 2.240 ms, Optimization 19.376 ms, Emission 12.317 ms, Total 34.491 ms
 Execution Time: 100927.097 ms
(11 строк)

postgres=#
```

_Использован Nested Loop_

_Полное соединение_

_Находим строки без соответствия_

## 1
```
postgres=# EXPLAIN ANALYZE
postgres-# SELECT a.buyer_id, b.buyer_id FROM customers A FULL JOIN purchases b ON a.buyer_id=b.buyer_id WHERE a.buyer_id IS null OR b.buyer_id IS null;
                                                                      QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------------------------------
 Merge Full Join  (cost=0.57..154458.33 rows=1 width=8) (actual time=2.308..1110.584 rows=35810 loops=1)
   Merge Cond: (a.buyer_id = b.buyer_id)
   Filter: ((a.buyer_id IS NULL) OR (b.buyer_id IS NULL))
   Rows Removed by Filter: 4964220
   ->  Index Only Scan using idx_cust_uid on customers a  (cost=0.14..13.64 rows=100 width=4) (actual time=0.015..0.313 rows=100 loops=1)
         Heap Fetches: 100
   ->  Index Only Scan using idx_pur_uid on purchases b  (cost=0.43..91944.43 rows=5000000 width=4) (actual time=0.025..589.393 rows=5000000 loops=1)
         Heap Fetches: 0
 Planning Time: 0.104 ms
 JIT:
   Functions: 4
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 0.373 ms, Inlining 0.000 ms, Optimization 0.217 ms, Emission 1.921 ms, Total 2.511 ms
 Execution Time: 1112.648 ms
(14 строк)

postgres=#
```

_Использован Merge Full Join_

_Ипользование разных типов соединений_

_Выполняем прямое содинение таблицы customers с самой собой и левостороннее соединение с таблицей purchases_

## 1
```
postgres=# EXPLAIN ANALYZE
postgres-# SELECT a.buyer_id FROM customers A INNER JOIN customers b ON a.buyer_id=b.buyer_id
postgres-# LEFT JOIN purchases c ON a.buyer_id=c.buyer_id;
                                                                      QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------------------------------
 Merge Left Join  (cost=776.31..154454.88 rows=5000000 width=4) (actual time=11.646..1291.409 rows=4964250 loops=1)
   Merge Cond: (a.buyer_id = c.buyer_id)
   ->  Sort  (cost=9.95..10.20 rows=100 width=4) (actual time=6.796..6.903 rows=100 loops=1)
         Sort Key: a.buyer_id
         Sort Method: quicksort  Memory: 25kB
         ->  Hash Join  (cost=3.25..6.62 rows=100 width=4) (actual time=6.770..6.790 rows=100 loops=1)
               Hash Cond: (a.buyer_id = b.buyer_id)
               ->  Seq Scan on customers a  (cost=0.00..2.00 rows=100 width=4) (actual time=0.021..0.026 rows=100 loops=1)
               ->  Hash  (cost=2.00..2.00 rows=100 width=4) (actual time=6.740..6.742 rows=100 loops=1)
                     Buckets: 1024  Batches: 1  Memory Usage: 12kB
                     ->  Seq Scan on customers b  (cost=0.00..2.00 rows=100 width=4) (actual time=6.713..6.722 rows=100 loops=1)
   ->  Index Only Scan using idx_pur_uid on purchases c  (cost=0.43..91944.43 rows=5000000 width=4) (actual time=0.015..606.043 rows=5000000 loops=1)
         Heap Fetches: 0
 Planning Time: 0.343 ms
 JIT:
   Functions: 15
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 2.061 ms, Inlining 0.000 ms, Optimization 0.491 ms, Emission 6.002 ms, Total 8.554 ms
 Execution Time: 1500.528 ms
(19 строк)

postgres=#
```

_Структура таблиц_

## 1
```
postgres=# \d customers
                         Таблица "public.customers"
 Столбец  |   Тип   | Правило сортировки | Допустимость NULL | По умолчанию
----------+---------+--------------------+-------------------+--------------
 buyer_id | integer |                    |                   |
 passwd   | text    |                    |                   |
Индексы:
    "idx_cust_uid" btree (buyer_id)

postgres=# \d purchases
                             Таблица "public.purchases"
     Столбец     |   Тип    | Правило сортировки | Допустимость NULL | По умолчанию
-----------------+----------+--------------------+-------------------+--------------
 id              | integer  |                    |                   |
 buyer_id        | integer  |                    |                   |
 purchase_date   | date     |                    |                   |
 status          | text     |                    |                   |
 some_txt        | text     |                    |                   |
 some_txt_lexeme | tsvector |                    |                   |
Индексы:
    "idx_ord_id" UNIQUE, btree (id)
    "idx_pur_id30" UNIQUE, btree (id) WHERE id < 30
    "idx_pur_purchase_date_status" btree (purchase_date, status)
    "idx_pur_uid" btree (buyer_id)
    "search_index_pur" gin (some_txt_lexeme)

postgres=#
```