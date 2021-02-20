# Как создать простой http-сервер с помощью HTTP::Daemon

Приведенных ниже примеров вполне достаточно, чтобы получить представление о принципах работы
с модулем <font color="#00aa00">HTTP::Daemon</font>, и об организации работы простого сервера.

Основной особенностью работы с <font color="#00aa00">HTTP::Daemon</font> является то, что функциональность сервера обеспечивается не самим <font color="#00aa00">HTTP::Daemon</font>, а массой дополнительных модулей: <font color="#00aa00">URI, HTTP::Request, HTTP::Message, HTTP::Headers, HTTP::Response</font>. И для создания даже простого сервера, требуется понимать каждый из них.

## Самый простой пример сервера

```perl
#!/usr/local/bin/perl

use HTTP::Daemon;
use HTTP::Status;

my $d = HTTP::Daemon->new(Timeout => 15) || die;
print "Please contact me at: <url:", $d, ">\n";

while (my $c = $d->accept) {
  $r = $c->get_request;
  if ($r) {
    $c->send_basic_header;
    $c->print("Content-Type: text/plain");
    $c->send_crlf;
    $c->send_crlf;
    $c->print("Ok\n");
  }
  $c->close;
  undef($c);
}
exit;
```

Сервер запускается, выводит строку адреса, по которому он готов принимать запросы.
<pre>
%perl daemon.pl
Please contact me at: <url:http://dev-lab.info:55190>
</pre>
Если обратиться по указанному адресу (браузером, скриптом), сервер примет запрос и вернет ответ. В данном примере при создании объекта, был задан таймаут. Если по истечении указанного количества секунд к серверу никто не обратится, он завершит свою работу.


## Обработка данных из URL-строки

Имеется некая структура данных. Требуется, чтобы сервер возвращал данные из этой структуры,
в зависимости от пришедшего запроса.

```perl
#!/usr/local/bin/perl

use CGI;
use HTTP::Daemon;
use HTTP::Status;

my $department = {
  'db_group' => {
    'Igor' => {
      'office' => 315,
      'off_phone' => 2341,
    },
  },
  'web_group' => {
    'Maria' => {
      'office' => 202,
      'off_phone' => 2417,
    },
    'Anastasia' => {
      'office' => 202,
      'off_phone' => 2419,
    },
  },
};

my $d = HTTP::Daemon->new(Timeout => 20) || die;

print "Please contact me at: <url:", $d->url, ">\n";

while (my $c = $d->accept) {
  $r = $c->get_request;
  if ($r) {

    $c->send_basic_header;
    $c->print("Content-Type: text/plain");
    $c->send_crlf;
    $c->send_crlf;

    # разбираем, чего ввел клиент в URL-строке
    my @url_params = split(/\//, $r->url->path);
    my $query_params = CGI->new($r->url->query)->Vars;
    # в зависимости от содержимого URL, определяем: что ответить клиенту
    if ($url_params[1] &amp;&amp; $query_params->{name} &amp;&amp; $query_params->{data}) {

      my $info = $department->{$url_params[1]}->{$query_params->{name}}->{$query_params->{data}};
      if ($info) {
        $c->send_status_line;
        $c->print($info."\n");
      } else {
        $c->send_status_line(404);
      }
    } elsif ($url_params[1] &amp;&amp; $query_params->{name}) {
      my $info = $department->{$url_params[1]}->{$query_params->{name}};
      if ($info) {
        $c->send_status_line;
        foreach (keys %{$info}) {
          $c->print($_.":".$info->{$_}."\n");
        }
      } else {
        $c->send_status_line(404);
      }
    } elsif ($url_params[1]) {
      my $info = $department->{$url_params[1]};
      if ($info) {
        $c->send_status_line;
        foreach (keys %{$info}) {
          $c->print($_."\n");
        }
      } else {
        $c->send_status_line(404);
      }
    } else {
      $c->send_status_line;
      foreach (keys %{$department}) {
        $c->print($_."\n");
      }
    }
  }
  $c->close;
  undef($c);
}
exit;
```

Клиент может обратиться к данному серверу, используя адреса типа:
<ul>
<li>http://dev-lab.info:56449/ - вывод списка всех групп департамента</li>
<li>http://dev-lab.info:56449/web_group/ - вывод списка сотрудников заданной группы</li>
<li>http://dev-lab.info:56449/web_group?name=Maria - вывод всей информации о сотруднике</li>
<li>http://dev-lab.info:56449/web_group?name=Maria&amp;data=off_phone - вывод запрошенной информации о сотруднике, например, номера внутреннего телефона или номера кабинета</li>
</ul>
Если запрашиваемые данные в хранилище отсутствуют - клиент получит статус ответа <font color="#00aa00">404</font>.

## Прием данных от клиента

### Прием данных

```perl
while (my $c = $d->accept) {
  my $request = $c->get_request;
  if ($request) {
    #...
    my $content = $r->content();
    #...
  }
  $c->close;
  undef($c);
}
```

### Прием данных из формы

Если серверу была передана форма, и был указан тип данных <font color="#00aa00">'application/x-www-form-urlencoded'</font>, сервер может получить эти данные следующим образом:

```perl
while (my $c = $d->accept) {
  my $request = $c->get_request;
  if ($request) {
    #...
    my %query_params = $request->url->query_form;
    #...
  }
  $c->close;
  undef($c);
}
```

Метод <font color="#00aa00">query_form()</font> вернет данные в виде хеша, где каждый ключ - это name поля формы, а значение - содержимое этого поля.

## Передача данных клиенту

### Вывод клиенту сообщений об ошибках

Можно выводить клиенту сообщения об ошибках с помощью метода <font color="#00aa00">send_error()</font>. Ответ будет выслан в html-формате.

```perl
$r = $c->get_request;
if ($r) {
  $c->send_error(404, 'Error, text of error');
}
```

### Передача клиенту файлов

Можно в ответ на запрос, предлагать клиенту скачать файл.

```perl
$r = $c->get_request;
if ($r) {
  $c->send_file_response("file.tsv");
}
```

### Ответ клиенту с помощью HTTP::Response

Можно сформировать и отправить ответ клиенту с помощью <font color="#00aa00">HTTP::Response</font>.

```perl
$r = $c->get_request;
if ($r) {
  my $res = HTTP::Response->new(200);
  $res->header('Content-Type' => 'text/html');
  $res->content("OK, all be fine");
  $c->send_response($res);
}
```


