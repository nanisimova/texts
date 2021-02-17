# Пример простого FastCGI-сервера и FastCGI-клиента

*FastCGI-сервер на основе POE::Component::FastCGI и FastCGI-клиент на основе FCGI::Client*

## Что такое CGI

CGI – это стандарт, который описывает, как сервер должен запускать CGI-скрипт, как должен передавать ему параметры HTTP-запроса, как CGI-скрипт должен передавать результаты своей работы серверу.

При работе с CGI, web-сервер для обработки каждого запроса создает новый процесс.

Чтобы передать CGI-скрипту информацию о поступившем запросе, web-сервер перед запуском скрипта создает для него специальные переменные окружения и записывает в них всю информацию о текущем запросе.

CGI-скрипт получает доступ к значениям всех переменных через функции операционной системы, тем самым CGI-скрипт получает исчерпывающую информацию о запросе. Тело запроса (если оно есть) поступает на <font color="#00aa00">STDIN</font> (стандартный поток ввода) скрипта и имеет размер - <font color="#00aa00">CONTENT_LENGTH</font> байт.

Если CGI-скрипт хочет послать что-то в ответ, то он отправляет все выводимые данные в <font color="#00aa00">STDOUT</font> скрипта. Необходимо обязательно указать CGI-заголовок (поля <font color="#00aa00">Content-Type</font>, <font color="#00aa00">Location</font>, <font color="#00aa00">Status</font>), пустую строку, которая отделяет заголовок от тела ответа, и само тело ответа, тип которого был указан в <font color="#00aa00">Content-Type</font>.

Web-сервер, получив через <font color="#00aa00">STDOUT</font> ответ от CGI-скрипта, формирует HTTP-ответ и отправляет его клиенту.


## Что такое FastCGI

В отличие от CGI, FastCGI использует постоянно запущенные процессы для обработки множества запросов.

CGI-программы взаимодействуют с сервером через <font color="#00aa00">STDIN</font> и <font color="#00aa00">STDOUT</font> запущенного процесса.

FastCGI-процессы используют для связи с сервером <font color="#00aa00">Unix Domain Sockets</font> или <font color="#00aa00">TCP/IP</font> . Это даёт следующее преимущество над обычными CGI-программами: FastCGI-программы могут быть запущены не только на этом же сервере, но и где угодно в сети. Также возможна обработка запросов несколькими FastCGI-процессами, работающими параллельно. Можно использовать несколько FastCGI-серверов, распределяя нагрузку между ними с помощью <font color="#00aa00">nginx</font> или <font color="#00aa00">lighttpd</font>.

После установления соединения FastCGI-процесса с web-сервером, между ними начинается обмен данными, с использованием простого протокола, решающего две задачи: организация двунаправленного обмена в рамках одного соединения (для эмуляции STDIN, STDOUT, STDERR) и организация нескольких независимых FastCGI-сессий в рамках одного соединения.

Все передаваемые данные оборачиваются в FastCGI-записи - единицу данных протокола. FastCGI-записи служат для организации двунаправленного обмена и мультиплексирования нескольких сессий в рамках одного соединения.

FastCGI-запись состоит из заголовка фиксированной длины, следующего за ним содержимого и выравнивающих данных переменной длины. Каждая запись содержит 7 элементов.

## Пример простого FastCGI-сервера на основе POE-компоненты - POE::Component::FastCGI

Что делает FastCGI-сервер? Он постоянно слушает заданный порт на предмет получения новых запросов от web-сервера, распаковывает запись и обрабатывает запрос. Потом отправляет ответ обратно. Web-серверу остается только добавить http-заголовки и отправить ответ клиенту.

В отличие от CGI, в данном случае нет необходимости постоянно создавать новый процесс и запускать скрипты для обработки запросов. Процесс уже создан и ожидает запросы по заданному адресу.

```perl
#!/usr/bin/perl

use POE;
use POE::Component::FastCGI;

POE::Component::FastCGI->new(
     Port => 1026,
     Handlers => [
        [ 'page' => \&page ],
        [ '/' => \&default ],
     ]
);

sub default {
     my($request) = @_;

     my $response = $request->make_response;
     $response->header("Content-type" => "text/html");
     $response->content("Default Page");
     $response->send;
}

sub page {
     my($request) = @_;

     my $response = $request->make_response;
     $response->header("Content-type" => "text/html");
     $response->content("Category pages");
     $response->send;
}

POE::Kernel->run;
```

## Пример простого FastCGI-клиента на основе модуля FCGI::Client

Подходит для проверки работоспособности вышеприведенного FastCGI-сервера.

```perl
#!/usr/bin/perl

use FCGI::Client;
use IO::Socket::INET;

my $sock = IO::Socket::INET->new(
     PeerAddr => '127.0.0.1',
     PeerPort => 1026,
) or die $!;

my $client = FCGI::Client::Connection->new( sock => $sock );
my ( $stdout, $stderr ) = $client->request(
     {
          REQUEST_METHOD => 'GET',
          REQUEST_URI => '/',
          # QUERY_STRING   => 'param1=value1',
     }, ''
);

print $stdout." ".$stderr;
```

После запуска скрипт выведет:
<pre>% perl fastcgiclient.pl
Status: 200
Content-Length: 12
Content-Type: text/html

Default Page
</pre>
Кроме <font color="#00aa00">REQUEST_METHOD</font>, <font color="#00aa00">REQUEST_URI</font> и <font color="#00aa00">QUERY_STRING</font>, при разборе поступившего запроса компонента <font color="#00aa00">POE::Component::FastCGI</font> учитывает значения переменных <font color="#00aa00">SERVER_NAME</font>, <font color="#00aa00">HTTPS</font> и <font color="#00aa00">HTTP_HOST</font>. Следует иметь это ввиду, пытаясь разобраться, почему в ответ на запрос иногда приходит совсем не то, что вы ожидали.
