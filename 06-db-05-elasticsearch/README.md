# Домашнее задание к занятию "5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [centos:7](https://hub.docker.com/_/centos) как базовый и 
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib`
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- возможно вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения

Далее мы будем работать с данным экземпляром elasticsearch.

Ответ:
1) Dockerfile

```
FROM centos:7

EXPOSE 9200 9300

ENV ES_HOME="/var/lib/elasticsearch" \
    ES_PATH_CONF="/var/lib/elasticsearch/config" \
    ES_VERSION="elasticsearch-8.6.2"

RUN export ES_HOME="/var/lib/elasticsearch" && \
    yum makecache && \
    yum -y install wget && \
    yum -y install vim && \
    yum -y install iproute && \
    yum -y install perl-Digest-SHA && \
    wget https://artifacts.elastic.co/downloads/elasticsearch/${ES_VERSION}-linux-x86_64.tar.gz && \
    wget https://artifacts.elastic.co/downloads/elasticsearch/${ES_VERSION}-linux-x86_64.tar.gz.sha512 && \
    shasum -a 512 -c ${ES_VERSION}-linux-x86_64.tar.gz.sha512 && \
    tar -xzf ${ES_VERSION}-linux-x86_64.tar.gz && \
    mv ${ES_VERSION} ${ES_HOME}


RUN useradd -m -u 1000 elasticsearch && \
    chown elasticsearch:elasticsearch -R ${ES_HOME} && \
    chmod o+x ${ES_HOME}

#COPY --chown=elasticsearch:elasticsearch config/* /var/lib/elasticsearch/config/

WORKDIR ${ES_HOME}

USER elasticsearch

CMD ["sh", "-c", "${ES_HOME}/bin/elasticsearch"]
```

2) Reference

https://hub.docker.com/repository/docker/nvrein/netology/general
docker pull nvrein/netology:elastic

3)
```
curl --cacert http_ca.crt -u elastic https://localhost:9200
Enter host password for user 'elastic':
{
  "name" : "80c1bbdba7e2",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "cKwqmqJdTNySeJQfpV7Sww",
  "version" : {
    "number" : "8.6.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "2d58d0f136141f03239816a4e360a8d17b6d8f29",
    "build_date" : "2023-02-13T09:35:20.314882762Z",
    "build_snapshot" : false,
    "lucene_version" : "9.4.2",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

Ответ:
1)

```
curl --cacert http_ca.crt -u elastic -X PUT "https://localhost:9200/ind-1" -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
curl --cacert http_ca.crt -u elastic -X PUT "https://localhost:9200/ind-2" -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 2,  "number_of_replicas": 1 }}'
curl --cacert http_ca.crt -u elastic -X PUT "https://localhost:9200/ind-3" -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 4,  "number_of_replicas": 2 }}'
```

2)

```
curl --cacert http_ca.crt -u elastic -X GET "https://localhost:9200/_cat/indices?v"
Enter host password for user 'elastic':
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   ind-1 Vd9c3aGaSTmcgPaUI8LxiA   1   0          0            0       225b           225b
yellow open   ind-3 blflSo4bT_2_N8h1p_7n4Q   4   2          0            0       900b           900b
yellow open   ind-2 pzMI6hyQSTie6khn6ZiJQg   2   1          0            0       450b           450b

```

3)

```
Cтатус green есть только у первого индекса так как у него нет реплик, остальные со статусом yellow так как этим индексам мы назначили реплики, но реплик на самом деле нет и кластер сообщает что их нужно будет по хорошему добавить.
```

4)

```
curl --cacert http_ca.crt -u elastic -X DELETE https://localhost:9200/ind-1?pretty
curl --cacert http_ca.crt -u elastic -X DELETE https://localhost:9200/ind-2?pretty
curl --cacert http_ca.crt -u elastic -X DELETE https://localhost:9200/ind-3?pretty

Enter host password for user 'elastic':
{
  "acknowledged" : true
}
```

## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`

Ответ:

1)

```
echo path.repo: "/var/lib/elasticsearch/snapshots" >> config/elasticsearch.yml
docker restart elastic
```

```
curl --cacert http_ca.crt -u elastic -X PUT https://localhost:9200/_snapshot/netology_backup?pretty -H 'Content-Type: application/json' -d' { "type": "fs", "settings": { "location": "/var/lib/elasticsearch/snapshots"}}'
Enter host password for user 'elastic':
{
  "error" : {
    "root_cause" : [
      {
        "type" : "security_exception",
        "reason" : "unable to authenticate user [elastic] for REST request [/_snapshot/netology_backup?pretty]",
        "header" : {
          "WWW-Authenticate" : [
            "Basic realm=\"security\" charset=\"UTF-8\"",
            "Bearer realm=\"security\"",
            "ApiKey"
          ]
        }
      }
    ],
    "type" : "security_exception",
    "reason" : "unable to authenticate user [elastic] for REST request [/_snapshot/netology_backup?pretty]",
    "header" : {
      "WWW-Authenticate" : [
        "Basic realm=\"security\" charset=\"UTF-8\"",
        "Bearer realm=\"security\"",
        "ApiKey"
      ]
    }
  },
  "status" : 401
}

docker exec -it elastic /var/lib/elasticsearch/bin/elasticsearch-setup-passwords interactive
******************************************************************************
Note: The 'elasticsearch-setup-passwords' tool has been deprecated. This       command will be removed in a future release.
******************************************************************************

Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]:
Reenter password for [elastic]:
...
Changed password for user [elastic]

curl --cacert http_ca.crt -u elastic -X PUT https://localhost:9200/_snapshot/netology_backup?pretty -H 'Content-Type: application/json' -d' { "type": "fs", "settings": { "location": "/var/lib/elasticsearch/snapshots"}}'
Enter host password for user 'elastic':
{
  "acknowledged" : true
}
```

2)

```
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test  sloMCdJETlWDY6zDkax1hA   1   0          0            0       225b           225b
```

3)

```
curl --cacert http_ca.crt -u elastic -X PUT "https://localhost:9200/_snapshot/netology_backup/snapshot_before?wait_for_completion=true&pretty"
Enter host password for user 'elastic':
{
  "snapshot" : {
    "snapshot" : "snapshot_before",
    "uuid" : "6Y41tbzDQS-zIrwQNYPUIQ",
    "repository" : "netology_backup",
    "version_id" : 8060299,
    "version" : "8.6.2",
    "indices" : [
      ".security-7",
      "test",
      ".geoip_databases"
    ],
    "data_streams" : [ ],
    "include_global_state" : true,
    "state" : "SUCCESS",
    "start_time" : "2023-03-18T04:28:52.737Z",
    "start_time_in_millis" : 1679113732737,
    "end_time" : "2023-03-18T04:28:56.568Z",
    "end_time_in_millis" : 1679113736568,
    "duration_in_millis" : 3831,
    "failures" : [ ],
    "shards" : {
      "total" : 3,
      "failed" : 0,
      "successful" : 3
    },
    "feature_states" : [
      {
        "feature_name" : "geoip",
        "indices" : [
          ".geoip_databases"
        ]
      },
      {
        "feature_name" : "security",
        "indices" : [
          ".security-7"
        ]
      }
    ]
  }
}

docker exec -it elastic ls -la /var/lib/elasticsearch/snapshots
total 48
drwxr-xr-x 3 elasticsearch elasticsearch  4096 Mar 18 04:28 .
drwxr-xr-x 1 elasticsearch elasticsearch  4096 Mar 18 04:29 ..
-rw-r--r-- 1 elasticsearch elasticsearch  1100 Mar 18 04:28 index-0
-rw-r--r-- 1 elasticsearch elasticsearch     8 Mar 18 04:28 index.latest
drwxr-xr-x 5 elasticsearch elasticsearch  4096 Mar 18 04:28 indices
-rw-r--r-- 1 elasticsearch elasticsearch 18841 Mar 18 04:28 meta-6Y41tbzDQS-zIrwQNYPUIQ.dat
-rw-r--r-- 1 elasticsearch elasticsearch   390 Mar 18 04:28 snap-6Y41tbzDQS-zIrwQNYPUIQ.dat
```

4)

```
curl --cacert http_ca.crt -u elastic -X DELETE https://localhost:9200/test?pretty                                                     
Enter host password for user 'elastic':
{
  "acknowledged" : true
}

curl --cacert http_ca.crt -u elastic -X PUT https://localhost:9200/test-2?pretty -H 'Content-Type: application/json' -d'{ "settings": { "index": { "number_of_shards": 1, "number_of_replicas": 0 }}}'
Enter host password for user 'elastic':
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test-2"
}

curl --cacert http_ca.crt -u elastic https://localhost:9200/_cat/indices?v                                                            
Enter host password for user 'elastic':
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 37boSl_fR86rUJKXRXDyeg   1   0          0            0       225b           225b
```

5)

```
curl --cacert http_ca.crt -u elastic -X POST "https://localhost:9200/_snapshot/netology_backup/snapshot_before/_restore?pretty"
Enter host password for user 'elastic':
{
  "accepted" : true
}
curl --cacert http_ca.crt -u elastic https://localhost:9200/_cat/indices?v                                                            
Enter host password for user 'elastic':
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 37boSl_fR86rUJKXRXDyeg   1   0          0            0       225b           225b
green  open   test   DYyQNf7oSoOgctktn_BGgA   1   0          0            0       225b           225b
```
---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
