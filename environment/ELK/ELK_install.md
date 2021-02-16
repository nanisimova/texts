# Установка Elasticsearch, Logstash и Kibana под Debian Linux

Установка Elasticsearch, Logstash и Kibana, с минимальными настройками, для работы в тестовом окружении. Установка проводилась под виртуальную машину с Debian Linux.

<pre>$ lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux 9.8 (stretch)
Release:        9.8
Codename:       stretch
</pre>
Все действия выполнялись от пользователя root (это ужасно, но на личном тестовом окружении почему бы и не позволить себе маленькие слабости, в т.ч. работу из-под root), поэтому sudo в приведенных примерах не используется.

## Установка Java

<pre>$ apt-get update
$ apt-get install openjdk-8-jdk
</pre>

Проверить версию установленной java:
<pre>$ java -version
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (build 1.8.0_212-8u212-b01-1~deb9u1-b01)
OpenJDK 64-Bit Server VM (build 25.212-b01, mixed mode)
</pre>

## Установка Elasticsearch на Debian
Elasticsearch - это поисковый сервер с открытым исходным кодом на основе Apache Lucene. Написан на Java, что позволяет Elasticsearch работать на разных платформах. Предоставляет RESTful web-API, хранит документы в JSON-формате без заранее определяемой структуры (schema less). Позволяет пользователям обрабатывать очень большой объем данных с очень высокой скоростью.

Копируем публичный ключ репозитория:
<pre>$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -</pre>

В файл /etc/apt/sources.list добавляем новую строку:
<pre>deb https://artifacts.elastic.co/packages/6.x/apt stable main</pre>

Приступаем к установке:
<pre>$ apt-get update
$ apt-get install elasticsearch
</pre>

Запустить Elasticsearch:
<pre>$ service elasticsearch start</pre>

Проверить состояние Elasticsearch:
<pre>$ service elasticsearch status</pre>

Конфигурационные файлы Elasticsearch хранятся в директории */etc/elasticsearch*.

Логи можно найти в директории */var/log/elasticsearch*.

Индексы и прочие необходимые данные elasticsearch будет размещать по адресу */var/lib/elasticsearch*. Местонахождение логов и индексов можно изменить, внеся правки в конфигурационные файлы.

Проверить, откликается elasticsearch или нет, можно отправив простой http-запрос:
<pre>$ curl -X GET http://localhost:9200</pre>

## Установка Kibana на Debian

Kibana - это web-интерфейс для визуализации данных Elasticsearch.
<pre>$ apt-get install kibana</pre>

Запускаем Kibana:
<pre>$ service kibana start</pre>

Проверить работоспособность Kibana можно, отправив запрос по адресу *http://localhost:5601*. Kibana запускается достаточно долго, поэтому не стоит нервничать, если сразу после выполнения команды start сервис продолжает оставаться недоступным.

Конфиг Kibana - */etc/kibana/kibana.yml*. Вспомогательные данные Kibana помещает в директорию */var/lib/kibana/*.

## Установка Logstash на Debian

Logstash принимает логи в различных форматах, обрабатывает и отправляет на индексацию Elasticsearch.
<pre>$ apt-get install logstash</pre>

Запускаем Logstash:
<pre>$ service logstash start</pre>

Конфигурационные файлы Logstash можно найти по адресу */etc/logstash/*. Логи - */var/log/logstash/*.

## Установка и настройка Nginx на Debian для работы с Kibana

Nginx выполняет роль прокси по отношению к Kibana. Nginx не является обязательным компонентом в тестовой среде, в начале изучения стека ELK. Но лучше сразу привыкать к его наличию. В реальных условиях nginx используют для добавления авторизации, ограничений доступа, ssl-сертификатов, управления доменными именами, и т.д.

Ниже приведен пример **минимальной конфигурации** nginx для работы с Kibana. Подобный конфиг подойдет на первое время для тестовой среды.

Устанавливаем nginx:
<pre>$ apt-get install nginx</pre>

Создаем новый файл **kibana** по адресу */etc/nginx/sites-available/*:
<pre>server {
  listen 9999;
  server_name kibana;

  error_log   /var/log/nginx/kibana.error.log;
  access_log  /var/log/nginx/kibana.access.log;

  location / {
    rewrite ^/(.*) /$1 break;
    proxy_ignore_client_abort on;
    proxy_pass http://localhost:5601;
    proxy_set_header  X-Real-IP  $remote_addr;
    proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header  Host $http_host;

  }
}
</pre>
Вместо указанного в примере порта 9999 можно назначить любой другой, главное, чтоб он не конфликтовал со стандартными портами Linux и другими используемыми на вашей машине серверами.

Создаем символическую ссылку:
<pre>$ ln -s /etc/nginx/sites-available/kibana /etc/nginx/sites-enabled/kibana</pre>

Запускаем nginx, если он еще не был запущен, либо перезагружаем:
<pre>$ service nginx start</pre>


