# Домашнее задание к занятию "ELK" - `Джансериков Эдуард`

---

### Задание 1

```
Установите и запустите Elasticsearch, после чего поменяйте параметр cluster_name на случайный.

Приведите скриншот команды 'curl -X GET 'localhost:9200/_cluster/health?pretty', сделанной на сервере с установленным Elasticsearch. Где будет виден нестандартный cluster_name.
```
![Задание 1](https://github.com/edjanserikov/redis/blob/main/img/Elastic.PNG)

---

### Задание 2

```
Установите и запустите Kibana.

Приведите скриншот интерфейса Kibana на странице http://<ip вашего сервера>:5601/app/dev_tools#/console, где будет выполнен запрос GET /_cluster/health?pretty.
```
![Задание 2](https://github.com/edjanserikov/redis/blob/main/img/Kibana.PNG)

---

### Задание 3

```
Установите и запустите Logstash и Nginx. С помощью Logstash отправьте access-лог Nginx в Elasticsearch.

Приведите скриншот интерфейса Kibana, на котором видны логи Nginx.
```
![Задание 3](https://github.com/edjanserikov/redis/blob/main/img/LogstashNginx.PNG)

---

### Задание 4

```
Установите и запустите Filebeat. Переключите поставку логов Nginx с Logstash на Filebeat.

Приведите скриншот интерфейса Kibana, на котором видны логи Nginx, которые были отправлены через Filebeat.
```
![Задание 4](https://github.com/edjanserikov/redis/blob/main/img/FilebeatNginx.PNG)

