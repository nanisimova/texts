# Как создать REST API на основе Catalyst

## Введение
### Что такое REST

REST - это принцип построения архитектуры программного обеспечения. Используется при разработке веб-сервисов.

### Что такое REST API

REST API - это набор функций, к которым могут обращаться разработчики. Используя HTTP-протокол, разработчик отправляет запрос и получает ответ. Как правило, данные передаются в одном из форматов: HTML, XML или JSON.

Запрос данных. Клиент обращается к веб-сервису, используя специальный URL. URL содержит все необходимые серверу данные, для однозначной идентификации запрашиваемого объекта. Методы HTTP-протокола используются для передачи информации о том, что мы хотим сделать с указанным объектом.

Например:
<ul>
<li>GET /articles — получить список всех публикаций</li>
<li>GET /article/1236 — получить из хранилища публикацию под номером 1236</li>
<li>PUT /article — добавить новую статью (данные в теле запроса)</li>
<li>POST /article/1236 – изменить статью (данные в теле запроса)</li>
<li>DELETE /article/33 – удалить публикацию</li>
</ul>

## Создаем REST API на основе Catalyst

### Краткое описание

Для реализации REST-сервиса мы будем использовать специальный модуль <font color="#00aa00">Catalyst::Controller::REST</font>, который значительно упрощает разработку соответствующих web-сервисов.

Catalyst::Controller::REST вмешивается в процесс диспетчеризации поступающих запросов.

Принцип диспетчиризации поступающих запросов изменяет добавление атрибута <font color="#00aa00">:ActionClass('REST')</font> к объявлению Catalyst action (м.б. кто-нибудь подскажет русскоязычный аналог этого термина?).

Например, если при объявлении метода <font color="#00aa00">article</font> указан <font color="#00aa00">:ActionClass('REST')</font>

```perl
sub article :Local :ActionClass('REST') {
    ...
}
```

то в дальнейшем, при обращении клиента по адресу <font color="#00aa00">/article</font> методом GET, Catalyst будет передавать
управление обработчику <font color="#00aa00">article_GET</font>, если использован метод POST, то обработчику <font color="#00aa00">article_POST</font>. Если соответствующий обработчик не найден - Catalyst вернет клиенту ответ со статусом - 405 (Method Not Found).

```perl
sub article_GET {
    ...
}
```

Другой вариант объявления метода, который будет вызываться для обработки запроса GET <font color="#00aa00">/article </font>

```perl
sub article_PUT : Action {
    ...
}
```

Если ответ со статусом 405 не нравится, и хочется отправить клиенту что-то особенное, можно переопределить метод, который вызывается для ответа в случае ошибки, и задать ему желаемое поведение:

```perl
sub article_not_implemented {
    ...
}
```

### Практическая реализация REST API на основе Catalyst. Примеры кода

Каркас приложения у нас уже создан. См. <a href="Catalyst_intro.md">"Как создать Catalyst-приложение с нуля"</a>.

Теперь добавим модули, которые еще не установлены, но понадобятся для создания REST API.

<pre>
force install Catalyst::View::JSON
force install Catalyst::Controller::REST
force install JSON::XS
</pre>

Создаем новое представление
<pre>perl script/myapp_create.pl view JSON JSON</pre>

Первый аргумент - название создаваемого представления, второй - название модуля, который мы наследуем при создании представления. Можно создавать представление, давая ему любое имя, например:
<pre>perl script/myapp_create.pl view MyJSONview JSON</pre>

, но идентичное названию модуля - мне кажется более удобным для дальнейшего использования.

Для работы мы будем использовать уже знакомую БД test (<a href="Catalyst_Controller_HTML_FormFu.md">см. дамп БД</a>) и таблицу статей.

API - это не только REST. Поэтому, для всех возможных API, я создаю отдельный каталог в директории Controller. Внутри API создаю каталог REST. Там будут все модули, которые отвечают за выполнение запросов.

#### Модуль /lib/MyApp/Controller/API/REST.pm

```perl
package MyApp::Controller::API::REST;

use Moose;
use namespace::autoclean;

BEGIN { extends 'Catalyst::Controller::REST' }

__PACKAGE__->config(
    default     => 'application/json',
);

sub rest_chain :Chained :PathPrefix :CaptureArgs(0) {}

__PACKAGE__->meta->make_immutable;

1;
```

#### Модуль /lib/MyApp/Controller/API/REST/Articles.pm

```perl
package MyApp::Controller::API::REST::Articles;

use Moose;
use namespace::autoclean;

BEGIN { extends 'MyApp::Controller::API::REST' }

=item articles

Action для всех запросов с URL /api/rest/articles

=cut

sub articles :Chained('../rest_chain') :PathPart('articles') :Args(0) 
:ActionClass('REST') {
    my ( $self, $c ) = @_;
}

=item articles_GET

Обработчик запросов /api/rest/articles, метод GET . Возвращает список публикаций

=cut

sub articles_GET {
    my ( $self, $c ) = @_;

    my $rs = $c->model('DB::Article')->search(
        {}
    );

    my %data;
    $data{count} = $rs->count;

    $data{articles}  = [
        map { +{
            id => $_->id,
            name => $_->name,
            full => $_->full
        } } $rs->all
    ];

    $self->status_ok(
        $c,
        entity => \%data
    );
}

=item article

Action для всех запросов с URL /api/rest/article/{id}

=cut

sub article :Chained('../rest_chain') :PathPart('article') :Args(1) 
:ActionClass('REST') {
    my ( $self, $c ) = @_;

}

=item article_GET

Обработчик запросов /api/rest/article/{id}, метод GET . Возвращает информацию 
о публикации под указанным id.

=cut

sub article_GET {
    my ( $self, $c, $id ) = @_;

    my $article = $c->model('DB::Article')->find(
        {'id' => $id}
    );

    my %data;
    if (defined $article) {

        $data{article}  = {
            id => $article->id,
            name => $article->name,
            full => $article->full
        };

    } else {

        $data{status} = '404';

    }

    $self->status_ok(
        $c,
        entity => \%data
    );
}

=item article_DELETE

Обработчик запросов /api/rest/article/{id}, метод DELETE . 
Удаляет из БД публикацию с указанным id.

=cut

sub article_DELETE {
    my ( $self, $c, $id ) = @_;

    my $article = $c->model('DB::Article')->find(
        {'id' => $id}
    );

    my %data;
    if (defined $article) {

        $article->delete();
        $data{status} = '200';

    } else {

        $data{status} = '404';

    }

    $self->status_ok(
        $c,
        entity => \%data
    );
}

=item article

Action для всех запросов с URL /api/rest/article

=cut

sub article :Chained('../rest_chain') :PathPart('article') :Args(0) 
:ActionClass('REST') {
    my ( $self, $c ) = @_;

}

=item article_PUT

Обработчик запросов /api/rest/article, метод PUT, тело запроса содержит данные 
для создания новой записи в БД, в формате {"name": "arrr", "full":"dadaa"}
 
=cut

sub article_PUT {
    my ($self, $c) = @_;

    my $args = $c->request->data;

    my %data;

    my $res = $c->model("DB::Article")->create({
        name => $args->{name} || '',
        full => $args->{full} || '',
    });

    $data{status} = '200';

    $self->status_ok(
        $c,
        entity => \%data
    );
}

__PACKAGE__->meta->make_immutable;

1;
```

Все данные клиенту будут возвращаться в JSON-формате.

#### Представление /lib/MyApp/View/JSON.pm

В представление нужно добавить обработчик "process":

```perl
package MyApp::View::JSON;

use strict;
use base 'Catalyst::View::JSON';

sub process {
    my($self, $c) = @_;
    $c->res->header('Pragma' => 'no-cache');
    $c->res->header('Cache-Control' => 'no-store, no-cache, must-revalidate, post-check=0, pre-check=0');
    $self->SUPER::process( $c, @_ );
    $c->res->header('Content-type'  => 'application/json; charset=utf-8');
}

1;
```

#### Проверяем работоспособность REST-сервисов

Для тестирования REST-сервисов использовала <b>RESTClient - специальный плагин для Firefox</b>. Очень удобно, рекомендую.

Чтобы получить список статей, надо в адресной строке web-клиента указать:
<pre>http://localhost:3000/api/rest/articles</pre>
и выполнить запрос методом GET.

Чтобы отправить PUT запрос с данными, нужно передавать данные через поле "Request Body", в JSON-формате. Например:
<pre>{"name": "arrr", "full":"dadaa"}</pre>

**Примечание:** Если REST API используется для внутренних нужд, можно ограничить доступ к нему настройками сервера. Если web-сервис должен быть доступен внешним клиентам, необходимо ввести для клиентов этапы авторизации и аутентификации.



