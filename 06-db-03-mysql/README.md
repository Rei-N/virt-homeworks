# Домашнее задание к занятию "3. MySQL"

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h` получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из ее вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с данным контейнером.

Ответ:
1) docker-compose
```
version: "3"

volumes:
  db:
    driver: local

services:

  db:
    image: mysql:8.0
    cap_add:
      - SYS_NICE
    restart: always
    ports:
      - 3306:3306
    container_name: mysql
    environment:
      - MYSQL_DATABASE=test_db
      - MYSQL_ROOT_PASSWORD=mysql
    volumes:
      - db:/var/lib/mysql
      - ./test_dump.sql:/docker-entrypoint-initdb.d/test_dump.sql
```

2)
```
mysql> \s
--------------
mysql  Ver 8.0.32 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:          11
Current database:       mysql
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         8.0.32 MySQL Community Server - GPL
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    latin1
Conn.  characterset:    latin1
UNIX socket:            /var/run/mysqld/mysqld.sock
Binary data as:         Hexadecimal
Uptime:                 13 min 28 sec

Threads: 2  Questions: 95  Slow queries: 0  Opens: 245  Flush tables: 3  Open tables: 164  Queries per second avg: 0.117
--------------
```

3)
```
cmysql> connect test_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Connection id:    12
Current database: test_db

mysql> show tables;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.00 sec)
```

4)
```
mysql> select count(id) from orders where price > 300;
+-----------+
| count(id) |
+-----------+
|         1 |
+-----------+
1 row in set (0.00 sec)
```

## Задача 2
Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
- срок истечения пароля - 180 дней 
- количество попыток авторизации - 3 
- максимальное количество запросов в час - 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James"

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

Ответ:
1)
```
mysql> create user 'test' identified with mysql_native_password by 'test-pass' with max_queries_per_hour 100 password expire interval 180 day failed_login_attempts 3 attribute '{"fname": "James","lname": "Pretty"}';
Query OK, 0 rows affected (0.02 sec)
```

2)
```
mysql> grant select on test_db.* to test;
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.02 sec)
```

3)
```
mysql> select * from INFORMATION_SCHEMA.USER_ATTRIBUTES where user = 'test';
+------+------+---------------------------------------+
| USER | HOST | ATTRIBUTE                             |
+------+------+---------------------------------------+
| test | %    | {"fname": "James", "lname": "Pretty"} |
+------+------+---------------------------------------+
1 row in set (0.01 sec)
```

## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`
- на `InnoDB`

Ответ:
1)
```
mysql> SET profiling=1;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

2)
```
mysql> SHOW PROFILES;
+----------+------------+-------------------+
| Query_ID | Duration   | Query             |
+----------+------------+-------------------+
|        1 | 0.01485075 | show table status |
|        2 | 0.01007500 | show table status |
+----------+------------+-------------------+
2 rows in set, 1 warning (0.00 sec)
```

3)
```
mysql> show table status\G;
*************************** 1. row ***************************
           Name: orders
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 2
 Avg_row_length: 8192
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 6
    Create_time: 2023-02-25 08:17:26
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options:
        Comment:
1 row in set (0.00 sec)
```

4)
```
mysql> SHOW PROFILES;
+----------+------------+----------------------------------+
| Query_ID | Duration   | Query                            |
+----------+------------+----------------------------------+
|        1 | 0.01485075 | show table status                |
|        2 | 0.01007500 | show table status                |
|        3 | 0.03604725 | ALTER TABLE orders ENGINE=MyISAM |
|        4 | 0.08484475 | ALTER TABLE orders ENGINE=InnoDB |
+----------+------------+----------------------------------+
4 rows in set, 1 warning (0.00 sec)
```

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):
- Скорость IO важнее сохранности данных
- Нужна компрессия таблиц для экономии места на диске
- Размер буффера с незакомиченными транзакциями 1 Мб
- Буффер кеширования 30% от ОЗУ
- Размер файла логов операций 100 Мб

Приведите в ответе измененный файл `my.cnf`.

Ответ:
Добавить эти строки
```
innodb_file_per_table = 1
innodb_file_format=Barracuda
innodb_flush_method = O_DIRECT
innodb_log_buffer_size=1M
innodb_log_file_size = 100M
innodb_buffer_pool_size = 2400M.
```


---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
