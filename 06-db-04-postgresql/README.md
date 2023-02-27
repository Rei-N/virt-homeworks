# Домашнее задание к занятию "4. PostgreSQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
- подключения к БД
- вывода списка таблиц
- вывода описания содержимого таблиц
- выхода из psql

Ответ:

1)
```
\l - список БД
\c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo} connect to new database (currently "postgres") - подключение к БД
\dt[S+] [PATTERN] - список таблиц
\d[S+]  NAME - описание содержимого таблиц
\q - выход из psql
```

## Задача 2

Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.

Ответ:

1)
```
postgres=# CREATE DATABASE test_database;
CREATE DATABASE
```

2)
```
postgres=# CREATE DATABASE test_database;
CREATE DATABASE
postgres=# \q
postgres@5369042875e1:/$ psql -f /tmp/pg_test_dump.sql test_database
SET
SET
SET
SET
SET
 set_config
------------

(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
COPY 8
 setval
--------
      8
(1 row)

ALTER TABLE
```

3)
```
test_database=# ANALYZE orders;
ANALYZE
```

4)
```
test_database=# select avg_width from pg_stats where tablename = 'orders';
 avg_width
-----------
         4
        16
         4
```

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

Ответ:
1)

```
test_database=# begin;
BEGIN
test_database=*# create table orders_partitioned (
        id integer NOT NULL,
        title varchar(80) NOT NULL,
        price integer) partition by range(price);
CREATE TABLE
test_database=*# create table orders_1 partition of orders_partitioned for values from (499) to (99999);
CREATE TABLE
test_database=*# create table orders_2 partition of orders_partitioned for values from (0) to (499);
CREATE TABLE
test_database=*# insert into orders_partitioned (id, title, price) select * from orders;
INSERT 0 8
test_database=*# commit;
COMMIT

test_database=# \dt
                     List of relations
 Schema |        Name        |       Type        |  Owner
--------+--------------------+-------------------+----------
 public | orders             | table             | postgres
 public | orders_1           | table             | postgres
 public | orders_2           | table             | postgres
 public | orders_partitioned | partitioned table | postgres
```

2) При изначальном проектировании таблицы нужно было заложить партицирование.

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
