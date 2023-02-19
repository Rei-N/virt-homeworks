# Домашнее задание к занятию "2. SQL"

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.

Ответ:

```
version: "3"

volumes:
  pg_data:
  pg_backup:

services:

  postgressql:
    image: postgres:12
    ports:
      - 5432:5432
    container_name: postgresql
    environment:
      - PGDATA=/var/lib/postgresql/data/
      - POSTGRES_PASSWORD=pgsql
    volumes:
      - pg_data:/var/lib/postgresql/data
      - pg_backup:/var/usr/backup

```

## Задача 2

В БД из задачи 1: 
- создайте пользователя test-admin-user и БД test_db
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
- создайте пользователя test-simple-user  
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db

Таблица orders:
- id (serial primary key)
- наименование (string)
- цена (integer)

Таблица clients:
- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)

Приведите:
- итоговый список БД после выполнения пунктов выше,
- описание таблиц (describe)
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
- список пользователей с правами над таблицами test_db

Ответ:
1)
```
  datname  | datcollate
-----------+------------
 postgres  | en_US.utf8
 test_db   | en_US.utf8
 template1 | en_US.utf8
 template0 | en_US.utf8
(4 rows)
```

2)
```
 table_catalog | table_schema | table_name | column_name  |     data_type
---------------+--------------+------------+--------------+-------------------
 postgres      | public       | orders     | id           | integer
 postgres      | public       | orders     | order_name   | character varying
 postgres      | public       | orders     | price        | integer
 postgres      | public       | clients    | id           | integer
 postgres      | public       | clients    | last_name    | character varying
 postgres      | public       | clients    | country      | character varying
 postgres      | public       | clients    | order_number | integer
(7 rows)
```

3)
```
SELECT grantee, table_name, privilege_type
FROM information_schema.table_privileges
WHERE grantee in ('test-admin-user','test-simple-user') and table_name in ('clients','orders') order by grantee;
```

4)
```
     grantee      | table_name | privilege_type
------------------+------------+----------------
 test-admin-user  | orders     | INSERT
 test-admin-user  | orders     | SELECT
 test-admin-user  | orders     | UPDATE
 test-admin-user  | orders     | DELETE
 test-admin-user  | orders     | TRUNCATE
 test-admin-user  | orders     | REFERENCES
 test-admin-user  | orders     | TRIGGER
 test-admin-user  | clients    | INSERT
 test-admin-user  | clients    | SELECT
 test-admin-user  | clients    | UPDATE
 test-admin-user  | clients    | DELETE
 test-admin-user  | clients    | TRUNCATE
 test-admin-user  | clients    | REFERENCES
 test-admin-user  | clients    | TRIGGER
 test-simple-user | clients    | INSERT
 test-simple-user | orders     | INSERT
 test-simple-user | orders     | SELECT
 test-simple-user | orders     | UPDATE
 test-simple-user | orders     | DELETE
 test-simple-user | clients    | SELECT
 test-simple-user | clients    | UPDATE
 test-simple-user | clients    | DELETE
(22 rows)
```

## Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL синтаксис:
- вычислите количество записей для каждой таблицы 
- приведите в ответе:
    - запросы 
    - результаты их выполнения.

Ответ:
1)
```
postgres=# select count(id) from orders;
 count
-------
     5
(1 row)

postgres=# select count(id) from clients;
 count
-------
     5
(1 row)
```

2)
```
postgres=# INSERT INTO orders VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
INSERT 0 5
postgres=# select * from orders;
 id | order_name | price
----+------------+-------
  1 | Шоколад    |    10
  2 | Принтер    |  3000
  3 | Книга      |   500
  4 | Монитор    |  7000
  5 | Гитара     |  4000
(5 rows)

postgres=# INSERT INTO clients VALUES (1, 'Иванов Иван Иванович', 'USA'), (2, 'Петров Петр Петрович', 'Canada'), (3, 'Иоганн Себастьян Бах', 'Japan'), (4, 'Ронни Джеймс Дио', 'Russia'), (5, 'Ritchie Blackmore', 'Russia');
INSERT 0 5
postgres=# select * from clients;
 id |      last_name       | country | order_number
----+----------------------+---------+--------------
  1 | Иванов Иван Иванович | USA     |
  2 | Петров Петр Петрович | Canada  |
  3 | Иоганн Себастьян Бах | Japan   |
  4 | Ронни Джеймс Дио     | Russia  |
  5 | Ritchie Blackmore    | Russia  |
(5 rows)
```

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения данных операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
 
Подсказк - используйте директиву `UPDATE`.

Ответ:
1)
```
postgres=# UPDATE clients SET "order_number" = (SELECT id FROM orders WHERE "order_name"='Книга') WHERE "last_name"='Иванов Иван Иванович';
UPDATE 1
postgres=# UPDATE clients SET "order_number" = (SELECT id FROM orders WHERE "order_name"='Монитор') WHERE "last_name"='Петров Петр Петрович';
UPDATE 1
postgres=# UPDATE clients SET "order_number" = (SELECT id FROM orders WHERE "order_name"='Гитара') WHERE "last_name"='Иоганн Себастьян Бах';
UPDATE 1
```

2)
```
postgres=# SELECT c.last_name, c.country, o.order_name  FROM clients c JOIN orders o ON c.order_number = o.id;
      last_name       | country | order_name
----------------------+---------+------------
 Иванов Иван Иванович | USA     | Книга
 Петров Петр Петрович | Canada  | Монитор
 Иоганн Себастьян Бах | Japan   | Гитара
(3 rows)
```

##Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

Ответ:
```
postgres=# explain SELECT c.last_name, c.country, o.order_name  FROM clients c JOIN orders o ON c.order_number = o.id;
                               QUERY PLAN
------------------------------------------------------------------------
 Hash Join  (cost=27.55..41.97 rows=350 width=264)
   Hash Cond: (c.order_number = o.id)
   ->  Seq Scan on clients c  (cost=0.00..13.50 rows=350 width=200)
   ->  Hash  (cost=17.80..17.80 rows=780 width=72)
         ->  Seq Scan on orders o  (cost=0.00..17.80 rows=780 width=72)
```
EXPLAIN - позволяет нам дать служебную информацию о запросе к БД, в том числе время на выполнение запроса, что при оптимизации работы БД является очень полезной информацией.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

Ответ:
```
root@efa3b17dbeb6:/# export PGPASSWORD=password && pg_dumpall -h localhost -U test-admin-user > /var/usr/backup/backup_db.sql

root@efa3b17dbeb6:/# ls /var/usr/backup/
backup_db.sql

docker-compose stop
docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

docker run --rm -d -e POSTGRES_USER=test-admin-user -e POSTGRES_PASSWORD=password -e POSTGRES_DB=test_db -v /home/vagrant/virt-video-code/docker-compose/backup/:/var/usr/backup --name postgresql2 postgres:12

docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
hrf5n34rgfu8   postgres:12   "docker-entrypoint.s…"   7 seconds ago   Up 5 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgresql2

docker exec -it hrf5n34rgfu8 bash
root@hrf5n34rgfu8:/# ls /var/usr/backup/
backup_db.sql
root@hrf5n34rgfu8:/# export PGPASSWORD=password && psql -h localhost -U test-admin-user -f /var/usr/backup/backup_db.sql test_db
```

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
