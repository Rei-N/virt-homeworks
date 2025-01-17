# Домашнее задание к занятию "6. Troubleshooting"

## Задача 1

Перед выполнением задания ознакомьтесь с документацией по [администрированию MongoDB](https://docs.mongodb.com/manual/administration/).

Пользователь (разработчик) написал в канал поддержки, что у него уже 3 минуты происходит CRUD операция в MongoDB и её 
нужно прервать. 

Вы как инженер поддержки решили произвести данную операцию:
- напишите список операций, которые вы будете производить для остановки запроса пользователя
- предложите вариант решения проблемы с долгими (зависающими) запросами в MongoDB

Ответ:

1) Использовать операцию db.currentOp() - returns a document that contains information on in-progress operations for the database instance.

```
db.currentOp(
   {
     "active" : true,
     "secs_running" : { "$gt" : 180 },
     "ns" : /^db1\./
   }
)
```

Полученное id процесса в поле "opid" : pr_id

```
db.killOp({pr_id})
```

- Устанавить ограничение по времени для операции maxTimeMS()
- Включить профайлер чтобы отловить медленные запросы
- Постороить/перестроить соответствующий индекс, денормализовать данные, настроить шардинг 


## Задача 2

Перед выполнением задания познакомьтесь с документацией по [Redis latency troobleshooting](https://redis.io/topics/latency).

Вы запустили инстанс Redis для использования совместно с сервисом, который использует механизм TTL. 
Причем отношение количества записанных key-value значений к количеству истёкших значений есть величина постоянная и
увеличивается пропорционально количеству реплик сервиса. 

При масштабировании сервиса до N реплик вы увидели, что:
- сначала рост отношения записанных значений к истекшим
- Redis блокирует операции записи

Как вы думаете, в чем может быть проблема?

Ответ:

1)
Вероятно случилось переполнение памяти истекшими ключами. Redis блокирует операции на запись, чтобы избежать утечки оперативной память и вернуть значение истекающих ключей до менее 25%(значение по умолчанию), путем очистки базы. О механизме описано в документации ниже

```
if the database has many, many keys expiring in the same second, and these make up at least 25% of the current population of keys with an expire set, Redis can block in order to get the percentage of keys already expired below 25%.

This approach is needed in order to avoid using too much memory for keys that are already expired
In short: be aware that many keys expiring at the same moment can be a source of latency
```

## Задача 3

Вы подняли базу данных MySQL для использования в гис-системе. При росте количества записей, в таблицах базы,
пользователи начали жаловаться на ошибки вида:
```python
InterfaceError: (InterfaceError) 2013: Lost connection to MySQL server during query u'SELECT..... '
```

Как вы думаете, почему это начало происходить и как локализовать проблему?

Какие пути решения данной проблемы вы можете предложить?

Ответ:

1)
Основываясь на документации MySQL https://dev.mysql.com/doc/лrefman/8.0/en/error-lost-connection.html

Возможны три причины:
1. Слишком объемные запросы на миллионы строк -> рекомендуется увеличение параметра net_read_timeout
2. Малое значение параметра connect_timeout, клиент не успевает установить соединение -> увеличить значение переменнйой connect_timeout
3. Размер сообщения/запроса превышает размер буфера max_allowed_packet на сервере или max_allowed_packet на строне клиента -> увеличить значение max_allowed_packet

Итого для рения проблемы можно:
Увеличить на сервере MySQL wait_timeout, connect_timeout, max_allowed_packet, net_write_timeout и net_read_timeout


## Задача 4


Вы решили перевести гис-систему из задачи 3 на PostgreSQL, так как прочитали в документации, что эта СУБД работает с 
большим объемом данных лучше, чем MySQL.

После запуска пользователи начали жаловаться, что СУБД время от времени становится недоступной. В dmesg вы видите, что:

`postmaster invoked oom-killer`

Как вы думаете, что происходит?

Как бы вы решили данную проблему?

Ответ:

1)
Причина "postmaster invoked oom-killer" в недостатке ресурсов оперативной памяти, чтобы предотвратить падение всей системы ОС Linux завершает процес утилизирующий память, 
посредством OOM(Out-Of-Memory) Killer.

```
1. По возможности добавить ресурсов ОЗУ 
2. Провести ревизию сервисов на предмет утечек памяти, неоптимизированном использовании.
3. Произвести настройку параметров, затрагивающих память в Postgres:
max_connections
shared_buffer
work_mem
effective_cache_size
maintenance_work_mem
4. В крайнмй случай можно произвести настройку параметров ОС в sysctl.conf, только при условии, что база данных не предоставляется как сервис.
vm.overcommit_memory
vm.dirty_ratio
vm.swappiness
```

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
