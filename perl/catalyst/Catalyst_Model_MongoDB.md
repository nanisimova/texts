# Использование MongoDB в Catalyst. Часть 1. Примеры кода

*Разработка панели администратора, для доступа к системе логов. Создание модели для работы с MongoDB в Catalyst. Создание контроллера и tt-шаблонов для панели администратора. Взаимодействие с MongoDB, выборка данных, сортировка, использование лимитов, поиск одной-единственной записи. Получение списка всех доступных баз данных Mongo. Использование Variable::Eject. Простые примеры кода.*

**Задача**

MongoDB используется для хранения логов. Требуется создать панель администратора, заходя в которую, администратор сможет просматривать все записи таблицы (сорри за терминологию, просто так привычнее) с логами. Для начала можно создать всего две страницы:
<ul>
<li>первая - это таблица с записями, 20 записей на одной странице</li>
<li>вторая - это подробная исформация о каждой записи.</li>
</ul>
В данных примерах не рассматривается удаление записей и создание новых, хотя на основе изложенного, это не сложно сделать самостоятельно. Создание системы логирования в данной публикации так же не рассматривается.

Приведенный код строится на ранее созданной основе:
<ul>
<li><a href="Catalyst_intro.md">Как создать catalyst-приложение с нуля</a></li>
<li><a href="Catalyst_Plugin_Static_Simple.md">Использование статики в catalyst-приложении</a></li>

<li><a href="Catalyst_api.md">Взаимодействие Catalyst-приложения с внешним API. Пример 1</a></li>
<li><a href="MooseX_Singleton.md">Использование MooseX::Singleton в Catalyst-приложении</a></li>
</ul>


### 1. Создаем модель для работы с MongoDB

*/lib/app/Model/MongoDB.pm* :

```perl
package app::Model::MongoDB;

use strict;
use uni::perl ':dumper';.
use MooseX::Singleton;
use base qw/Catalyst::Model::MongoDB/;

__PACKAGE__->config(
    'host' => '127.0.0.1',
    'port' => '27017',
    'dbname' => 'app_db',
    'username' => 'app_user',
    'password' => 'app_pass',
);

1;
```

<font color="#00aa00">Catalyst::Model::MongoDB</font> используется в качестве базового класса для создания модели <font color="#00aa00">MongoDB</font>. Требуется только задать параметры подключения к БД, и модель можно использовать.


### 2. Создаем контроллер

Контроллер будет заниматься обработкой двух страниц - вывод основного списка логов, и вывод информации о конкретной записи в логах. Так же, будет реализована возможность перехода по страницам, хотя в шаблоне tt визуально это пока никак не оформлено.

Возможно, создание страницы для отображения конкретной записи в данном примере не оправдано, т.к. там выводятся все те же данные, что и в основной таблице. Однако, в дальнейшем страница будет полезна для добавления на нее формы редактирования записей, полного списка полей (которых может быть намного больше, чем в приведенном примере), комментариев и дополнительной информации.

*/lib/app/Controller/Log/ApiLog.pm* :

```perl
package app::Controller::Log::ApiLog;

use uni::perl ':dumper';
use Moose;
use namespace::autoclean;

use app::Model::MongoDB;
use Variable::Eject;

BEGIN { extends 'Catalyst::Controller' }

sub index :Path('/log/api_log') :Args(0) {
    my ( $self, $c ) = @_;

    eject ($c->req->params, $page, $limit);
    $page ||= 0;
    $limit ||= 20;

    my $db = app::Model::MongoDB->instance;

    # узнаем список всех баз данных Mongo на сервере
    my @dbnames = $db->collnames;

    my $coll = $db->coll('network_log');
    my $logs = $coll->find->sort({'id' => 1})->limit(10)->skip($page * $limit);

    my @res;
    my $i = 1;
    my @headers;
    while (my $log = $logs->next) {

        if ($i==1){
            # простой способ узнать, какие столбцы есть в строке
            foreach my $key (keys %{$log}) {
                push @headers, $key;
            }
        $i++;
        }

        push @res, {
            id => $log->{'_id'},
            source_id => $log->{source_id},
            response => $log->{response},
            ctime => $log->{ctime},
        };
    }

    $c->stash->{dbnames} = \@dbnames;
    $c->stash->{logs} = \@res;
    $c->stash->{fields} = \@headers;

    $c->stash->{template} = 'log/api_log/index.tt';
}

sub element :Path('/log/api_log') :Args(1) {
    my ( $self, $c, $id ) = @_;

    $c->stash->{template} = 'log/api_log/element.tt';

    my $db = app::Model::MongoDB->instance;
    my $coll = $db->coll('network_log');

    my $row = $coll->find_one({_id => MongoDB::OID->new(value => $id)});
    $row->{id} = $row->{_id};

    $c->stash->{row} = $row;
    $c->stash->{template} = 'log/api_log/element.tt';
}

__PACKAGE__->meta->make_immutable;

1;
```


### Использование Variable::Eject

<font color="#00aa00">Variable::Eject</font> - отличный модуль, который позволяет извлекать из указанного хеша значения для переменных. Предварительно объявлять переменные нет необходимости.

```perl
use Variable::Eject;
# из хеша с данными $c->req->params будут извлечены значения переданных 
# параметров page и limit, и присвоены одноименным переменным.
eject ($c->req->params, $page, $limit);
```


### 3. Создаем шаблон для главной страницы со списком логов

*/root/src/log/api_log/index.tt* :

```html
<link rel="stylesheet" type="text/css" href="/static/css/admin.css">
Список всех БД на mongo:
[% FOREACH name = dbnames %]
    [% name %]
[% END %]

Полный список полей network_logs:
[% FOREACH field = fields %]
    [% field %]
[% END %]

[% FOREACH row = logs %]
    [% do_something %]    
[% END %]

<table border="1">
<tbody>
<tr>
<td>id</td>
<td>source_id</td>
<td>ctime</td>
<td>response</td>
</tr>
<tr>
<td><a href="/log/api_log/[% row.id %]">[% row.id %]</a></td>
<td>[% row.source_id %]</td>
<td>[% row.ctime %]</td>
<td>[% row.response %]</td>
</tr>
</tbody>
</table>
```

### 4. Создаем шаблон для странички с полным описанием одной записи из таблицы логов

*/root/src/log/api_log/element.tt*:

```html
<link rel="stylesheet" type="text/css" href="/static/css/admin.css">

<table border="1">
<tbody>
<tr>
<td>id</td>
<td>[% row.id %]>/td></td>
</tr>
<tr>
<td>source_id</td>
<td>[% row.source_id %]</td>
</tr>
<tr>
<td>ctime</td>
<td>[% row.ctime %]</td>
</tr>
<tr>
<td>response</td>
<td>[% row.response %]</td>
</tr>
</tbody>
</table>
```

<h3>5. Создаем файл css</h3>
Страницы панели администратора выглядят просто ужасно. Для того, чтобы облагородить их вид, можно использовать CSS. Я привела только начало файла (просто для того, чтобы обозначить появление данного файла в текущих доработках кода), остальное можно дописать по собственному усмотрению.

*/root/static/css/admin.css* :

```html
body {
    margin: 0;
    padding: 10px;
    font-size: 12px;
    font-family: Verdana, Arial;
    color: #333;
    background: #fff;
}

a:link {
    color: #5b80b2;
    text-decoration: none;
}

a:hover {
    color: #003f6f;
}
```

### 6. Запускаем сервер и проверяем работоспособность кода

Запуск сервера. Для разнообразия - под портом 3001.

<pre>perl script/app_server.pl --port 3001</pre>

Главная страница, которая содержить список из 20 последних записей в БД Mongo:

<pre>http://localhost:3001/log/api_log</pre>

Использование параметров page и <font color="#00aa00">limit</font> позволяет организовать постраничный просмотр. По-умолчанию, <font color="#00aa00">page</font> имеет значение 0 и открывает первую страницу, а <font color="#00aa00">limit</font> имеет значение 20.

<pre>http://localhost:3001/log/api_log?page=2&amp;limit=30</pre>

Страница с подробным содержимым конкретной записи в БД. На эту же страницу можно добавить формы для редактирования данных.

<pre>http://localhost:3001/log/api_log/53330e8c3476bc2fe0000333</pre>

