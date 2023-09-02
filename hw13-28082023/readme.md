# Домашнее задание к лекции "Хранимые функции и процедуры часть 3"
## Дата: 28.08.2023
## Тема: Триггеры, поддержка заполнения витрин
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ alse174cmdt3, IP 192.168.186.173, экран  1, (2 core, 4 Gb, 20 Gb), ОС AstraLinux 1.7.4 mode 0, ПО Тантор (СУБД 15)
## Описание/Пошаговая инструкция выполнения домашнего задания:
_Скрипт и развернутое описание задачи – в ЛК (файл hw_triggers.sql) или по ссылке: https://disk.yandex.ru/d/l70AvknAepIJXQ
В БД создана структура, описывающая товары (таблица goods) и продажи (таблица sales).
Есть запрос для генерации отчета – сумма продаж по каждому товару.
БД была денормализована, создана таблица (витрина), структура которой повторяет структуру отчета.
Создать триггер на таблице продаж, для поддержки данных в витрине в актуальном состоянии (вычисляющий при каждой продаже сумму и записывающий её в витрину)_

## Выполнение

_Создаем демонстрационную БД_

## 1
```
postgres=# DROP SCHEMA IF EXISTS pract_functions CASCADE;
ЗАМЕЧАНИЕ:  схема "pract_functions" не существует, пропускается
DROP SCHEMA
postgres=# CREATE SCHEMA pract_functions;
CREATE SCHEMA
postgres=# SET search_path = pract_functions, publ;
SET
```

_Товары: Создаём таблицы с продажами, товарами и витриной_

## 1
```
postgres=# CREATE TABLE goods
postgres-# (
postgres(#     goods_id    integer PRIMARY KEY,
postgres(#     good_name   varchar(63) NOT NULL,
postgres(#     good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
postgres(# );
CREATE TABLE
postgres=# INSERT INTO goods (goods_id, good_name, good_price) VALUES (1, 'Спички хозайственные', .50),(2, 'Автомобиль Ferrari FXX K', 185000000.01);
INSERT 0 2
```

_Продажи_

## 1
```
postgres=# CREATE TABLE sales
postgres-# (
postgres(#     sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
postgres(#     good_id     integer REFERENCES goods (goods_id),
postgres(#     sales_time  timestamp with time zone DEFAULT now(),
postgres(#     sales_qty   integer CHECK (sales_qty > 0)
postgres(# );
CREATE TABLE
```

_Отчет_

## 1
```
postgres=# SELECT G.good_name, sum(G.good_price * S.sales_qty)
postgres-# FROM goods G
postgres-# INNER JOIN sales S ON S.good_id = G.goods_id
postgres-# GROUP BY G.good_name;
 good_name | sum
-----------+-----
(0 строк)
```

_С увеличением объёма данных отчет стал создаваться медленно_
_Принято решение денормализовать БД, создать таблицу_

## 1
```
postgres=# CREATE TABLE good_sum_mart(good_name varchar(63) NOT NULL, sum_sale numeric(16, 2)NOT NULL);
CREATE TABLE
```

_Инициализация_

## 1
```
postgres=# INSERT INTO good_sum_mart (good_name, sum_sale) VALUES ('Спички хозайственные',0),('Автомобиль Ferrari FXX K',0);
INSERT 0 2
```

_Создать триггер (на таблице sales) для поддержки._
_Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE_

## 1
```
postgres=# CREATE OR REPLACE FUNCTION public.edit_mart()
postgres-#  RETURNS trigger
postgres-#  LANGUAGE plpgsql
postgres-# AS $function$
postgres$# BEGIN
postgres$# IF    TG_OP = 'INSERT' THEN
postgres$# update good_sum_mart SET sum_sale = sum_sale + (SELECT good_price from goods WHERE NEW.good_id = goods.goods_id) * NEW.sales_qty
postgres$# WHERE good_name = (SELECT good_name FROM goods WHERE NEW.good_id = goods.goods_id);
postgres$# RETURN NEW;
postgres$# ELSIF TG_OP = 'DELETE' THEN
postgres$# update good_sum_mart SET sum_sale = sum_sale - (SELECT good_price from goods WHERE OLD.good_id = goods.goods_id) * OLD.sales_qty
postgres$# WHERE good_name = (SELECT good_name FROM goods WHERE OLD.good_id = goods.goods_id);
postgres$# RETURN OLD;
postgres$# ELSIF TG_OP = 'UPDATE' THEN
postgres$# update good_sum_mart SET sum_sale = sum_sale - (SELECT good_price from goods WHERE NEW.good_id = goods.goods_id) * OLD.sales_qty
postgres$# WHERE good_name = (SELECT good_name FROM goods WHERE OLD.good_id = goods.goods_id);
postgres$# update good_sum_mart SET sum_sale = sum_sale + (SELECT good_price from goods WHERE NEW.good_id = goods.goods_id) * NEW.sales_qty
postgres$# WHERE good_name = (SELECT good_name FROM goods WHERE NEW.good_id = goods.goods_id);
postgres$# RETURN NEW;
postgres$# END IF;
postgres$# END
postgres$# $function$;
CREATE FUNCTION
postgres=# create trigger tr_edit_mart BEFORE INSERT OR DELETE OR UPDATE ON sales FOR EACH ROW EXECUTE FUNCTION public.edit_mart ();
CREATE TRIGGER
```

_Проверяем_

## 1
```
postgres=# INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);
INSERT 0 4
postgres=# SELECT * FROM good_sum_mart ;
        good_name         |   sum_sale
--------------------------+--------------
 Спички хозайственные     |        65.50
 Автомобиль Ferrari FXX K | 185000000.01
(2 строки)
```

_Триггер отработал, добавил машину и спички_

## 1
```
postgres=# select * from sales;
 sales_id | good_id |          sales_time           | sales_qty
----------+---------+-------------------------------+-----------
        1 |       1 | 2023-09-02 22:54:18.216192+03 |        10
        2 |       1 | 2023-09-02 22:54:18.216192+03 |         1
        3 |       1 | 2023-09-02 22:54:18.216192+03 |       120
        4 |       2 | 2023-09-02 22:54:18.216192+03 |         1
(4 строки)

postgres=# DELETE FROM sales WHERE sales_id = 1;
DELETE 1
postgres=# select * from good_sum_mart ;
        good_name         |   sum_sale
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозайственные     |        60.50
(2 строки)

```

_Имитируем продажу_

## 1
```
postgres=# UPDATE sales SET sales_qty = 2 WHERE sales_time = '2023-09-02 22:54:18.216192+03';
UPDATE 3
postgres=# select * from sales;
 sales_id | good_id |          sales_time           | sales_qty
----------+---------+-------------------------------+-----------
        2 |       1 | 2023-09-02 22:54:18.216192+03 |         2
        3 |       1 | 2023-09-02 22:54:18.216192+03 |         2
        4 |       2 | 2023-09-02 22:54:18.216192+03 |         2
(3 строки)

postgres=# select * from good_sum_mart ;
        good_name         |   sum_sale
--------------------------+--------------
 Спички хозайственные     |         2.00
 Автомобиль Ferrari FXX K | 370000000.02
(2 строки)
```

_Перечитываем все продажи за 02.09.2023_


_Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?_
_Подсказка: В реальной жизни возможны изменения цен._

_Предположительно в витрине отразится актуальная информация о продажах, при условии отсутствия внесений изменений в sales после_