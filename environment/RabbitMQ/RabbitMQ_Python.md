# Работа с RabbitMQ в Python. С чего начать

## Протокол AMQP

AMQP (Advanced Message Queueing Protocol) - открытый протокол для передачи сообщений между компонентами системы с низкой задержкой, на высокой скорости, а так же с высоким уровнем надежности и безопасности. Обмен сообщениями осуществляется через AMQP-брокер.

Популярные брокеры, поддерживающие AMQP: RabbitMQ (Erlang), Qpid (C++, Java), ActiveMQ (Java), StormMQ (Java), JORAM (Java).

Ключевые понятия AMQP:
<ul>
<li>Message - сообщение.</li>
<li>Queue - очередь. Принимает и сохраняет сообщение, пока оно не будет затребовано клиентом.</li>
<li>Exchange - точка обмена, сортировочный пункт. Принимает и распределяет сообщения в одну или несколько очередей. Поведение точки обмена зависит от типа:
<ul>
<li>fanout — входящее сообщение передается во все связанные с точкой очереди.</li>
<li>direct — входящее сообщение передается в очередь, имя которой указано в параметре routing key. Routing key (ключ маршрутизации) - указывается при отправке сообщения.</li>
<li>topic — входящее сообщение передается в очередь или несколько очередей по маске, указанной в routing key.</li>
</ul>
</li>
</ul>
В настоящее время AMQP использует в качестве транспортного протокола TCP. В будущем предполагается добавить поддержку работы с SCTP. Для AMQP официально выделен порт 5672 (TCP, UDP, SCTP) и порт 5671 для AMQPS (AMQP с шифрованием TLS/SSL).

## Что такое RabbitMQ

RabbitMQ - это брокер сообщений. Поддерживает протоколы AMQP, HTTP, STOMP, MQTT и веб-сокеты, WebSockets, Web-Stomp.

Изначально RabbitMQ создавался для работы с AMQP, который считается "основным" протоколом RabbitMQ.

Надежность - одно из важнейших достоинств RabbitMQ. RabbitMQ можно настроить так, чтобы сообщения всегда доставлялись.

### Доступ к портам RabbitMQ

Для взаимодействия с RabbitMQ необходимо знать связанные с ним номера портов, по которым будут ожидаться запросы. Однако, в некоторых случаях брандмауэры и другие инструменты безопасности могут блокировать привязку RabbitMQ к тому или иному порту. В этом случае RabbitMQ не запустится. Следует помнить об этом.
<ul>
<li>1883, 8883: клиенты MQTT и MQTTS (с шифрованием TLS/SSL), если включен плагин MQTT</li>
<li>4369: порт epmd - аналога DNS для Erlang-кластера. Epmd используется узлами RabbitMQ и инструментами CLI (CLI - интерфейс командной строки).</li>
<li>5672, 5671: используется AMQP и AMQPS (с шифрованием TLS/SSL) клиентами. 5672 - основной порт RabbitMQ. Доступа к нему достаточно для полноценной работы любых сервисов с очередью сообщений.</li>
<li>15672: для версии RabbitMQ 4.x клиенты HTTP API и RabbitMQadmin (если включен плагин управления). Иными словами, веб-интерфейс к RabbitMQ.</li>
<li>15674: клиенты STOMP-over-WebSockets, если включен плагин Web STOMP.</li>
<li>15675: клиенты MQTT-over-WebSockets, если включен плагин Web MQTT.</li>
<li>25672: используется для межсетевых интерфейсов и коммуникационных средств CLI. Выделяется из динамического диапазона, номер порта вычисляется как порт AMQP + 20000.</li>
<li>61613, 61614: клиенты STOMP и STOMPS (с шифрованием TLS/SSL), если включен плагин STOMP.</li>
</ul>

### Терминология RabbitMQ

Основные понятия RabbitMQ:
<ul>
<li>Producer — клиент, который создает сообщения.</li>
<li>Consumer — клиент, который получает сообщения.</li>
<li>Queue — неограниченная по размеру очередь, которая хранит сообщения.</li>
<li>Exchange — компонент, который позволяет переправлять отправляемые в него сообщения на различные очереди.</li>
</ul>
Есть значительно более эффективные системы обмена сообщениями, чем RabbitMQ, например Apache Kafka, ZeroMQ, механизм PUB/SUB в Redis, NATS.

### Запуск RabbitMQ с помощью Docker

RabbitMQ совместим не со всеми версиями Linux Debian, поэтому я использовала Docker. Для целей обучения и тестирования - этого вполне достаточно.

Запускаем RabbitMQ:
<pre>docker run --hostname localhost -p 8080:5672 -p 15672:15672 rabbitmq:3-management</pre>

Я сопоставила порт 8080 на своей машине и порт RabbitMQ - 5672. Это означает, что мои скрипты будут обращаться по адресу localhost:8080 для того, чтоб получить или отправить сообщение в очередь. Вы можете назначить другие номера портов.

Вторая связка - 15672:15672 - для сопоставления порта RabbitMQ, используемого для обращения к веб-интерфейсу и порта 15672 на локальной машине. Для доступа к веб-интерфейсу в строке браузера потребуется указать адрес http://localhost:15672.

Если при запуске контейнера порты не указать - обращение к RabbitMQ окажется невозможным.

По умолчанию для доступа к RabbitMQ используются логин/пароль: guest/guest .

<img src="rabbitmq_webinterface.png" alt="RabbitMQ web-интерфейс" class="size-full wp-image-2231" width="684" height="695">

## RabbitMQ и python. Модуль pika

В Python есть несколько известных библиотек, которые позволяют работать с RabbitMQ и протоколом AMQP: py-amqplib, txAMQP, pika.

Я выбрала модуль pika.

Pika предоставляет набор объектов для работы с RabbitMQ:
<ul>
<li>Connection adapter (соединение с RabbitMQ),</li>
<li>Connection parameters (параметры соединения),</li>
<li>Authentication Credentials (аутентификационные данные),</li>
<li>Channel (связь с RabbitMQ через RPC-методы AMQP),</li>
<li>Exceptions.</li>
</ul>

### Connection adapter

Pika позволяет использовать несколько адаптеров для установления соединения с RabbitMQ:
<ul>
<li>**Tornado Connection Adapter**.</li>
<li>**Twisted Connection Adapter**.</li>
<li>**Select Connection Adapter** - асинхронный адаптер соединения, который пытается использовать самый быстрый адаптер цикла событий для данной платформы.</li>
<li>**BlockingConnection** - модуль адаптера блокирующего соединения реализует блокирующую семантику поверх основного драйвера AMQP. Пользовательские классы в модуле состоят из классов BlockingConnection и BlockingChannel. Создание BlockingChannel:
<pre>
import pika

connection = pika.BlockingConnection()
channel = connection.channel()
</pre>
</li>
</ul>

### Аутентификация

PlainCredentials и ExternalCredentials - классы учетных данных. Инкапсулируют аутентификационные данные для класса ConnectionParameters.

Класс **PlainCredentials** возвращает имя пользователя и пароль для подключения в требуемом формате.

Для аутентификации с помощью Pika создается объект PlainCredentials, с именем пользователя и паролем в качестве аргументов. Созданный объект передается в качестве значения аргумента credentials объекту ConnectionParameters. Если при создании объекта ConnectionParameters логин и пароль не указаны, будут использованы значения по умолчанию: логин "guest" с паролем "guest".

Если используется URLParameters, объект учетных данных будет создан автоматически.

Класс **ExternalCredentials** позволяет при подключении к RabbitMQ использовать внешнюю аутентификацию, как правило, с клиентским SSL-сертификатом.

### Параметры подключения

Чтобы обеспечить гибкость в указании информации о коннекте к RabbitMQ, pika реализует два класса для инкапсуляции информации: ConnectionParameters и URLParameters.
<ul>
<li>**ConnectionParameters** - классический объект для указания всех параметров подключения, необходимых для подключения к RabbitMQ. ConnectionParameters предоставляет атрибуты для настройки всех возможных параметров подключения.
<pre>
import pika

credentials = pika.PlainCredentials('guest', 'guest')
parameters = pika.ConnectionParameters('rabbit-server1', 5672, '/', credentials)
</pre>
</li>
<li>**URLParameters** - класс позволяет передавать URL-адрес AMQP при создании объекта и поддерживает хост, порт, виртуальный хост, ssl, имя пользователя и пароль в базовом URL-адресе, а другие параметры передаются через параметры запроса.
<pre>import pika

parameters = pika.URLParameters('amqp://guest:guest@rabbit-server1:5672/%2F')
</pre>
</li>
</ul>

### Connection

Класс Connection реализует базовое поведение, которое расширяют все адаптеры подключения.

Это основной класс, который реализует связь с RabbitMQ. Этот класс должен вызываться не напрямую, а через использование адаптера, такого как SelectConnection или BlockingConnection.

### Channel

Класс Channel предоставляет оболочку для взаимодействия с RabbitMQ, реализуя методы для канала AMQP.

Канал является основным методом связи для взаимодействия с RabbitMQ. Рекомендуется создавать канал посредством вызова метода channel() для активного соединения.

### Примеры использования pika

#### Извлечь из очереди сообщение
```python
import pika

credentials = pika.PlainCredentials('guest', 'guest')
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost', 8080, '/', credentials))

channel = connection.channel()
method_frame, header_frame, body = channel.basic_get('my_queue')
if method_frame:
    print(method_frame, header_frame, body)
    channel.basic_ack(method_frame.delivery_tag)
else:
    print('No messages')
```

#### Поместить сообщение в очередь

```python
import pika

credentials = pika.PlainCredentials('guest', 'guest')
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost', 8080, '/', credentials))
channel = connection.channel()

'''
Если очередь с указанным именем не существует, queue_declare() создаст ее
'''
channel.queue_declare(queue='my_queue')
'''
Имя очереди должно быть определено в параметре routing_key
'''
channel.basic_publish(exchange='', routing_key='my_queue', body="Hello World!")
connection.close()
```

