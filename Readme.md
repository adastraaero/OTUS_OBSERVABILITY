## Домашнее задание 1

<details>
Установка и настройка Prometheus, использование exporters

Цель:

Установить и настроить Prometheus.
Результатом выполнения данного ДЗ будет являться публичный репозиторий, в системе контроля версий (Github, Gitlab, etc.), в котором будет находиться Readme с описанием выполненых действий.

Описание/Пошаговая инструкция выполнения домашнего задания:

Файлы конфигурации prometheus и alertmanager должны находиться в директории GAP-1.

Описание ДЗ:

На виртуальной машине установите любую open source CMS, которая включает в себя следующие компоненты: nginx, php-fpm, database (MySQL or Postgresql).
На этой же виртуальной машине установите Prometheus exporters для сбора метрик со всех компонентов системы (начиная с VM и заканчивая DB, не забудьте про blackbox exporter, который будет проверять доступность вашей CMS).
На этой же или дополнительной виртуальной машине установите Prometheus, задачей которого будет раз в 5 секунд собирать метрики с экспортеров.

### Решение:
Устанавливаем CMS Wordpress с использованием MYSQL, Nginx, Php-fpm.

![Image 1](HomeWork_1/Images/sc2.png)

Устанавливаем Prometheus и необходимые экспортеры.

Пишем юниты для экспортеров и добавляем их в автозагрузку.

Настраиваем экспортеры и проверяем, через prometheus.

![Image 2](HomeWork_1/Images/sc1.png)

</details>


## Домашнее задание 5

### Задача

Настройка zabbix, создание LLD, оповещение на основе триггеров

Цель:
Установить и настроить zabbix, настроить автоматическую отправку аллертов в телеграмм канал.


Описание/Пошаговая инструкция выполнения домашнего задания:
Необходимо сформировать скрипт генерирующий метрики формата:

```
otus_important_metrics[metric1]
otus_important_metrics[metric2]
otus_important_metrics[metric3]

```

С рандомным значение от 0 до 100

Создать правила LLD для обнаружения этих метрик и автоматического добавления триггеров. Триггер должен срабатывать тогда, когда значение больше или равно 95.

Реализовать автоматическую отправку уведомлений в телеграмм канал.

В качестве результаты выполнения ДЗ необходимо предоставить скрипт генерации метрик, скриншоты графиков полученных метрик, ссылку на телеграмм канал с уже отпраленными уведомлениями.


Решение:

<details>

На пк с установленным zabbix client в папке /etc/zabbix/zabbix_agentd.conf.d создаём файл lld.conf c содержимым:

```
UserParameter=otus.discovery,/tmp/sender_test.sh

```

Создаём в папке /tmp/ скрипт с содержимым sender_test.sh


```
#!/bin/bash

# send back discovery key, list of all available array keys
# for a discovery type of "Zabbix agent"
cat << EOF
{ "data": [
  { "{#ITEMNAME}":"otus_important_metrics1" },
  { "{#ITEMNAME}":"otus_important_metrics2" },
  { "{#ITEMNAME}":"otus_important_metrics3" }
]}
EOF

# now take advantage of this invocation to send back values
# build up list of values in /tmp/zdata.txt
agenthost="ubt-wp"
zserver="172.17.50.101"
zport="10051"

cat /dev/null > /tmp/zdata.txt
for item in "otus_important_metrics1" "otus_important_metrics2" "otus_important_metrics3"; do
  randNum="$(( (RANDOM % 100)+1 ))"
  echo $agenthost warning[$item] $randNum >> /tmp/zdata.txt
done

# push all these trapper values back to zabbix
zabbix_sender -vv -z $zserver -p $zport -i /tmp/zdata.txt >> /tmp/zsender.log 2>&1


```

Создаём в папке tmp файлы zdata.txt и zsender.log :

```
touch /tmp/zdata.txt
touch /tmp/zsender.log
```


Назначем владельцев и права на файлы:

```
chown zabbix:zabbix /tmp/z*.*
chmod 664 /tmp/z*.*
chown zabbix:zabbix /tmp/produce.sh
chmod 755 /tmp/produce.sh
```

Проверяем выполнение скрипта:
```
su -c "/tmp/sender_test.sh" -s /bin/sh zabbix
```

Создаём шаблон otus.lld на сервере zabbix

![](/HomeWork_5/img/zb1.png)


Добавляем в шаблон правило обнаружения

![](/HomeWork_5/img/zb3.png)

Создаём в шаблоне item prototype

![](/HomeWork_5/img/zb3.png)


Добавляем шаблон к хосту ubt-wp(ubuntu 22.04 server)

Проверяем что метрики меняются

![](/HomeWork_5/img/zb4.png)
![](/HomeWork_5/img/zb5.png)
![](/HomeWork_5/img/zb6.png)


Настраиваем warnings с разными уровнями значимости и проверяем что они работают:

![](/HomeWork_5/img/zb8.png)
![](/HomeWork_5/img/zb9.png)
![](/HomeWork_5/img/zb7.png)

Подключение телеграм бота

Находим в ТГ @BotFather и командой /newbot создаём нового бота, сохраняем его токен,
добавляем его в наш список контактов.

У бота @getmyid_bot получаем наш ID(chat_id).

Проверяем, что бот отправляет уведомления:

```
curl --header 'Content-Type: application/json' --request 'POST' --data '{"chat_id":"наш_чат_id","text":"Проверочное сообщение"}' "https://api.telegram.org/имя_бота:токен_бота/sendMessage"
```

Alerts - media types - настраиваем оповещение через ТГ
![](/HomeWork_5/img/zb10.png)


Alerts - Actios - trigger action - включаем уведомление для администраторов


![](/HomeWork_5/img/zb11.png)

Alerts - Actios - trigger action - Operations - настраиваем оповещение администраторов через ТГ.


![](/HomeWork_5/img/zb12.png)

Проверяем ТГ.

![](/HomeWork_5/img/tg1.png)


Скриншоты графиков полученных метрик

![](/HomeWork_5/img/zb13.png)
![](/HomeWork_5/img/zb14.png)
![](/HomeWork_5/img/zb15.png)




</details>

