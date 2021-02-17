# Пример простого web-клиента на основе perl и POE. Модуль POE::Component::Client::HTTP

## Пример web-клиента

```perl
#!/usr/bin/perl

use warnings;
use strict;

# используем HTTP::Request::Common для быстрого создания объектов HTTP::Request
use HTTP::Request::Common qw(GET POST);

my @url_list = qw(
  http://mysite.ru/test.html
);

use POE qw(Component::Client::HTTP);

POE::Component::Client::HTTP->spawn(
  Alias   => 'ua',
  Timeout => 180,
);

foreach my $url (@url_list) {
  # для каждого url создаем свою сессию
  POE::Session->create(
    inline_states => {
      _start => sub {
        my ($kernel, $heap) = @_[KERNEL, HEAP];

	$kernel->post("ua", "request", "got_response", GET $url);
      },
      # задаем обработчик для разбора http-ответа
      got_response => sub {
        my ($heap, $request_packet, $response_packet) = @_[HEAP, ARG0, ARG1];

        my $http_request = $request_packet->[0];
        my $http_response = $response_packet->[0];

        my $response_string = $http_response->as_string();
        print $response_string;
      },
    },
  );
}

$poe_kernel->run();
exit 0;
```

Алгоритм: для каждого url из списка создается своя сессия. Как только все сессии завершат свою работу, скрипт завершается. При этом, все сессии используют одну и ту же сессию POE::Component::Client::HTTP user-agent.

Подобное разделение запросов по отдельным сессиям является удобным при создании ботов, когда обращение к интернет-ресурсу является только одним этапом из целого списка последовательных операций.

## Описание POE::Component::Client::HTTP

<font color="#00aa00">POE::Component::Client::HTTP</font> - это HTTP user-agent для POE. Позволяет другим сессиям POE выполняться, пока идет обработка HTTP-транзакций. Данный подход позволяет выполняться сразу нескольким HTTP-запросам.

Компоненты POE::Component::Client::HTTP не являются соответствующими объектами. POE::Component::Client::HTTP порождает не объекты, а сессии. Именно поэтому вместо <font color="#00aa00">new</font> используется <font color="#00aa00">spawn</font>.

Вновь созданная сессия способна обработать несколько событий: <font color="#00aa00">request</font>, <font color="#00aa00">pending_requests_count</font>, <font color="#00aa00">cancel</font>, <font color="#00aa00">shutdown</font>

Для обращения к сессии POE::Component::Client::HTTP, POE использует метод <font color="#00aa00">post</font>.

Пример:
```perl
$kernel->post( ua => request, 'response', $request );
```
Указывается название сессии, имя вызываемого события (одно из встроенных в POE::Component::Client::HTTP), имя события, которое будет вызвано после выполнения запроса (задается разработчиком), и ссылка на объект HTTP::Request. Возможно добавление к указанному списку других параметров.

### request

При вызове данного события, выполняется запрос к заданному web-ресурсу.

### pending_requests_count

```perl
my $count = $kernel->call('ua' => 'pending_requests_count');
```
Возвращает количество обрабатываемых в текущий момент запросов.

### cancel

```perl
$kernel->post( component => cancel => $http_request );
```
Отменяет http-запрос. При этом следует дать ссылку на оригинал запроса, т.к. компонента обрабатывает множество запросов и не знает - какой конкретно отменить.

### shutdown

Все ожидающие обработки запросы отрубает по таймауту, завершает работу сессии и всех ее дочерних порождений.

## Как писать обработчик ответа на запрос

В дополнение ко всем доступным параметрам POE, HTTP-ответ возвращается с двумя списками ссылок.
```perl
my ($request_packet, $response_packet) = @_[ARG0, ARG1];
```

<font color="#00aa00">$request_packet</font> - содержит ссылку на оригинальный объект HTTP::Request.

```perl
my $http_request_object = $request_packet->[0];
```

<font color="#00aa00">$response_packet</font> - содержит ссылку на объект HTTP::Response, с результатами выполнения запроса.

```perl
my $http_response_object = $response_packet->[0];
```


