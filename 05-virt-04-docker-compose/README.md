# Домашнее задание к занятию "4. Оркестрация группой Docker контейнеров на примере Docker Compose"

## Как сдавать задания

Обязательными к выполнению являются задачи без указания звездочки. Их выполнение необходимо для получения зачета и диплома о профессиональной переподготовке.

Задачи со звездочкой (*) являются дополнительными задачами и/или задачами повышенной сложности. Они не являются обязательными к выполнению, но помогут вам глубже понять тему.

Домашнее задание выполните в файле readme.md в github репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.

Любые вопросы по решению задач задавайте в чате учебной группы.

---


## Важно!

Перед отправкой работы на проверку удаляйте неиспользуемые ресурсы.
Это важно для того, чтоб предупредить неконтролируемый расход средств, полученных в результате использования промокода.

Подробные рекомендации [здесь](https://github.com/netology-code/virt-homeworks/blob/virt-11/r/README.md)

---

## Задача 1

Создать собственный образ  любой операционной системы (например, ubuntu-20.04) с помощью Packer ([инструкция](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/packer-quickstart))

Для получения зачета вам необходимо предоставить скриншот страницы с созданным образом из личного кабинета YandexCloud.

Ответ: 
```
root@server1:/home/vagrant/virt-video-code/terraform# packer build ubuntu22.04.json
yandex: output will be in this color.

==> yandex: Creating temporary RSA SSH key for instance...
==> yandex: Using as source image: fd89n8278rhueakslujo (name: "ubuntu-22-04-lts-v20230123", family: "ubuntu-2204-lts")
==> yandex: Use provided subnet id e9bprgj9j4uqcoc14lff
==> yandex: Creating disk...
==> yandex: Creating instance...
==> yandex: Waiting for instance with id fhmo15vsapm1dr8nc39p to become active...
    yandex: Detected instance IP: 51.250.71.226
==> yandex: Using SSH communicator to connect: 51.250.71.226
==> yandex: Waiting for SSH to become available...
==> yandex: Connected to SSH!
==> yandex: Stopping instance...
==> yandex: Deleting instance...
    yandex: Instance has been deleted!
==> yandex: Creating image: ubuntu-2204-base
==> yandex: Waiting for image to complete...
==> yandex: Success image create...
==> yandex: Destroying boot disk...
    yandex: Disk has been deleted!
Build 'yandex' finished after 2 minutes 33 seconds.

==> Wait completed after 2 minutes 33 seconds

==> Builds finished. The artifacts of successful builds are:
--> yandex: A disk image was created: ubuntu-2204-base (id: fd83v0pbdmbk8c8v53au) with family name ubuntu-base
```
![image](https://user-images.githubusercontent.com/5601009/215697928-e00f884f-15f2-4ad3-91a2-c9186506eb00.png)

## Задача 2

Создать вашу первую виртуальную машину в YandexCloud с помощью terraform. 
Используйте terraform код в директории ([src/terraform](https://github.com/netology-group/virt-homeworks/tree/virt-11/05-virt-04-docker-compose/src/terraform))

Для получения зачета, вам необходимо предоставить вывод команды terraform apply и страницы свойств созданной ВМ из личного кабинета YandexCloud.

Ответ:
```
yandex_compute_instance.worker[0]: Creating...
yandex_compute_instance.manager[0]: Creating...
yandex_compute_instance.worker[0]: Still creating... [10s elapsed]
yandex_compute_instance.manager[0]: Still creating... [10s elapsed]
yandex_compute_instance.worker[0]: Still creating... [20s elapsed]
yandex_compute_instance.manager[0]: Still creating... [20s elapsed]
yandex_compute_instance.worker[0]: Still creating... [30s elapsed]
yandex_compute_instance.manager[0]: Still creating... [30s elapsed]
yandex_compute_instance.worker[0]: Still creating... [40s elapsed]
yandex_compute_instance.manager[0]: Still creating... [40s elapsed]
yandex_compute_instance.manager[0]: Still creating... [50s elapsed]
yandex_compute_instance.worker[0]: Still creating... [50s elapsed]
yandex_compute_instance.worker[0]: Creation complete after 51s [id=fhm2se97e2l2uenudrjd]
yandex_compute_instance.manager[0]: Creation complete after 52s [id=fhmh9fo5pm80s7vi1sj3]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

![image](https://user-images.githubusercontent.com/5601009/215698076-a530cb6b-279c-4b36-b35b-49ef6b0c742f.png)

## Задача 3

С помощью ansible и docker-compose разверните на виртуальной машине из предыдущего задания систему мониторинга на основе Prometheus/Grafana .
Используйте ansible код в директории ([src/ansible](https://github.com/netology-group/virt-homeworks/tree/virt-11/05-virt-04-docker-compose/src/ansible))

Для получения зачета вам необходимо предоставить вывод команды "docker ps" , все контейнеры, описанные в ([docker-compose](https://github.com/netology-group/virt-homeworks/blob/virt-11/05-virt-04-docker-compose/src/ansible/stack/docker-compose.yaml)),  должны быть в статусе "Up".

Ответ:
![image](https://user-images.githubusercontent.com/5601009/215698496-bb7b184a-29fa-4598-b37a-e98caade5ff4.png)


## Задача 4

1. Откройте веб-браузер, зайдите на страницу http://<внешний_ip_адрес_вашей_ВМ>:3000.
2. Используйте для авторизации логин и пароль из ([.env-file](https://github.com/netology-group/virt-homeworks/blob/virt-11/05-virt-04-docker-compose/src/ansible/stack/.env)).
3. Изучите доступный интерфейс, найдите в интерфейсе автоматически созданные docker-compose панели с графиками([dashboards](https://grafana.com/docs/grafana/latest/dashboards/use-dashboards/)).
4. Подождите 5-10 минут, чтобы система мониторинга успела накопить данные.

Для получения зачета, вам необходимо предоставить: 
- Скриншот работающего веб-интерфейса Grafana с текущими метриками, как на примере ниже
<p align="center">
  <img width="1200" height="600" src="./assets/yc_02.png">
</p>

Ответ:
![image](https://user-images.githubusercontent.com/5601009/215698653-0266edfe-2b00-411b-b37d-59062f0be3fa.png)


## Задача 5 (*)

Создать вторую ВМ и подключить её к мониторингу развёрнутому на первом сервере.

Для получения зачета, вам необходимо предоставить:
- Скриншот из Grafana, на котором будут отображаться метрики добавленного вами сервера.

