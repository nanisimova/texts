# Пример простого SOAP-сервера на основе компоненты POE::Component::Server::SOAP и SOAP-клиента на основе SOAP::Lite

## SOAP-сервер

Компонента <font color="#00aa00">POE::Component::Server::SOAP</font> для реализации работы с SOAP использует достаточно популярный модуль <font color="#00aa00">SOAP::Lite</font>.

В момент вызова метода <font color="#00aa00">new()</font>, <font color="#00aa00">POE::Component::Server::SOAP</font> создает новую сессию POE. Сразу после создания сессии запускается сервер, на основе компоненты <font color="#00aa00">POE::Component::Server::SimpleHTTP</font>.

Пример кода SOAP-сервера:

```perl
#!/usr/bin/perl

use POE;
use POE::Component::Server::SOAP;

POE::Component::Server::SOAP->new(
    'ALIAS'         =>  'MySOAP',
    'ADDRESS'       =>  'localhost',
    'PORT'          =>  32080,
    'HOSTNAME'      =>  'Localhost.com',
);

POE::Session->create(
  'inline_states' =>      {
    '_start'        =>  \&setup_service,
    '_stop'         =>  \&shutdown_service,
    'SOAP_Handler'  =>  \&soap_handler,
},
);

$poe_kernel->run;
exit 0;

sub setup_service {
  my $kernel = $_[KERNEL];
  $kernel->alias_set( 'MyServer' );
  $kernel->post( 'MySOAP', 'ADDMETHOD', 'MyServer', 'SOAP_Handler');
}

sub shutdown_service {
  $_[KERNEL]->post( 'MySOAP', 'DELMETHOD', 'MyServer', 'SOAP_Handler' );
}

sub soap_handler {
  my $response = $_[ARG0];
  my $params = $response->soapbody;

  # выводит поступивший запрос, удобно для логирования
  # print $response->soaprequest->as_string;

  # просто считывает все поступившие данные и собирает их
  # в одну строку
  my $result;
  foreach (values %$params) {
    $result= $result." | ".$_;  
  }

  $response->content( "Your request was processed: $result" );
  $_[KERNEL]->post( 'MySOAP', 'DONE', $response );

}
```

После запуска этого сервера можно быстро проверить его работоспособность, набрав в строке браузера <font color="#00aa00">http://localhost:32080/</font> . В ответ на запрос, сервер выдаст ответ:

```xml
<soap:envelope soap:encodingstyle="http://schemas.xmlsoap.org/soap/encoding/">
  <soap:body>
  <soap:fault>
    <faultcode>soap:Client</faultcode>
    <faultstring>Bad Request</faultstring>
    <detail xsi:type="xsd:string">Content-Type must be text/xml</detail>
  </soap:fault>
  </soap:body>
</soap:envelope>
```

## SOAP-клиент, вариант 1

Пример кода SOAP-клиента для вышеприведенного SOAP-сервера:

```perl
#!/usr/bin/perl

use strict;

use SOAP::Lite;

my $soap = SOAP::Lite
  ->uri('http://localhost:32080/')
  ->proxy('http://localhost:32080/?session=MyServer')
  ->outputxml('true');

my $res_xml = $soap->call('SOAP_Handler', 'Value1', 'Value2');

print $res_xml;
```

SOAP-клиент формирует вот такой запрос и отправляет его серверу:

```xml
POST http://Localhost.com:32080/?session=MyServer HTTP/1.1
Connection: TE, close
Accept: text/xml
Accept: multipart/*
Accept: application/soap
Host: localhost:32080
TE: deflate,gzip;q=0.3
User-Agent: SOAP::Lite/Perl/0.715
Content-Length: 530
Content-Type: text/xml; charset=utf-8
SOAPAction: "http://localhost:32080/#SOAP_Handler"

<!--?xml version="1.0" encoding="UTF-8"?-->
<soap:envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" soap:encodingstyle="http://schemas.xmlsoap.org/soap/encoding/" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:body>
    <soap_handler xmlns="http://localhost:32080/">
      <c-gensym3 xsi:type="xsd:string">Value1</c-gensym3>
      <c-gensym5 xsi:type="xsd:string">Value2</c-gensym5>
    </soap_handler>
  </soap:body>
</soap:envelope>
```

В ответ получает вот такой XML-документ:
```perl
% perl soapclient.pl
<!--?xml version="1.0" encoding="UTF-8"?-->
<soap:envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" soap:encodingstyle="http://schemas.xmlsoap.org/soap/encoding/" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:body> <namesp1:soap_handlerresponse="" xmlns:namesp1="http://localhost:32080/">
      <s-gensym3 xsi:type="xsd:string">
        Your request was processed:  | Value2 | Value1
      </s-gensym3>
    
  
</soap:body></soap:envelope>
```

Данный SOAP-клиент использует в качестве параметра <font color="#00aa00">outputxml('true')</font>, который приводит к тому, что <font color="#00aa00">SOAP::Lite</font> возвращает ответ от сервера как чистый xml-код. Если данная опция не задана, будет возвращен SOM-объект (см. модуль <font color="#00aa00">SOAP::SOM</font>).

Пример простого SOAP-клиента с использованием SOM см. далее.

## SOAP-клиент, вариант 2

Данный клиент использует в своей работе объект <font color="#00aa00">SOAP::SOM</font>.

```perl
#!/usr/bin/perl

use Data::Dumper;
use SOAP::Lite;

my $som = SOAP::Lite
  ->uri('http://localhost:32080/')
  ->proxy('http://localhost:32080/?session=MyServer')
  ->SOAP_Handler('Value1', 'Value2');

if ($som->fault) {
  print $som->faultstring." ".$som->faultdetail." ".$som->faultactor."\n";
};

print $som->result."\n";
```

Клиент формирует к серверу точно такой же запрос, как в предыдущем примере, и выводит на консоль ответ:
<pre>% perl soapclient2.pl
Your request was processed:  | Value2 | Value1
</pre>



