# Catalyst, Sphinx и realtime индекс

*Как создать realtime индекс Sphinx. Использование realtime индексов в Catalyst-приложении.*

Эксперименты проводились на Sphinx версий 2.2.10 и 2.0.4.

Продолжаем развивать код из <a href="Catalyst_Sphinx.md">первой заметки про Sphinx</a>. Возьмем его за основу и добавим в Catalyst использование realtime индекса.

## Конфигурация Sphinx и Catalyst-приложения для создания realtime индекса

В данном случае, для коннекта мы будем использовать не Sphinx::Search, а DBI.

/www/myapp/conf/sphinx/searchd.conf :
<pre>
searchd {
    listen = 127.0.0.1:9300:mysql41
    log = /var/log/sphinxsearch/searchd.log
    binlog_path = /var/lib/sphinxsearch/data
    query_log = /var/log/query.log
    pid_file = /var/run/sphinxsearch/searchd.pid
    read_timeout = 5
    client_timeout = 300
    max_children = 0
    max_matches = 1000000
    seamless_rotate = 1
    preopen_indexes = 1
    unlink_old = 1
    mva_updates_pool = 1M
    max_packet_size = 8M
    workers = threads
}
</pre>
Создаем описание realtime индекса. Список допустимых типов полей и атрибутов в realtime сильно отличается от тех,
которые используются в обычных индексах. Типов полей меньше и они более "требовательные".

/www/myapp/conf/sphinx/users_rt.conf:
<pre>
index users_rt {
    type=rt
    rt_attr_string = login
    rt_field = first_name
    rt_field = last_name
    rt_field = company

    docinfo = extern
    min_stemming_len = 2
    min_word_len = 2
    min_prefix_len = 2
    html_strip = 1
    charset_table = 0..9, A..Z-&gt;a..z, _, a..z, U+410..U+42F-&gt;U+430..U+44F, U+430..U+44F
    path = /var/data/sphinx/users_rt
    blend_chars = .
}
</pre>

/etc/sphinxsearch/sphinx.conf :
<pre>
#!/bin/bash

path="/www/myapp/conf/sphinx"
exec /usr/bin/env cat $path/sources.conf $path/searchd.conf $path/indexer.conf
$path/users.conf $path/users_rt.conf
</pre>

Если все настройки произведены верно, то при запуске сервера Sphinx будет создан realtime индекс.

## Добавление данных в realtime индекс

Теперь необходимо наполнить данными новый индекс. Существует 2 основных пути:
<ul>
<li>использовать ATTACH INDEX,</li>
<li>заполнение с помощью INSERT INTO и REPLACE INTO.</li>
</ul>

### ATTACH INDEX

ATTACH INDEX позволяет взять заполненный дисковый индекс и перенести все данные в realtime индекс.

Преимущества этого метода:
<ul>
<li>Очень быстро создается готовый к использованию realtime индекс.</li>
<li>Если по каким-то причинам realtime индекс был испорчен и требуется срочно восстановить - ATTACH INDEX позволяет сделать это намного быстрее, чем по одной строке добавлять данные.</li>
</ul>

Недостатки:
<ul>
<li>Следует быть очень аккуратным при подготовке переноса данных. Возможны самые неожиданные проблемы. Не все типы полей совместимы. Например, если дисковый индекс содержал boolean-атрибуты, а realtime вместо них использует integer (потому что, boolean-тип в RT отсутствует, по крайней мере, в нужной мне версии Sphinx его еще не было), то перенос пройдет "благополучно". Вы сможете даже извлекать данные из заполненного индекса. Но при попытке добавить новые строки вас ждет сюрприз. Таких нюансов в Sphinx - тысячи.</li>
<li>На этапе разработки использование ATTACH INDEX удобно только на первый взгляд. Вы создали индекс, радостно заполнили его данными. Дисковый индекс при этом обнулился.
А вам буквально через полчаса требуется изменить структуру RT индекса. В современных версиях Sphinx появилась возможность изменять структуру с помощью ALTER TABLE. Однако, на той версии, на которой тестировала свои наработки я, такой возможности еще нет, чтобы внести изменения в индекс надо полностью удалить все файлы которые к нему относятся, после чего пересобрать все заново. Заполнить данными второй раз не получится - дисковый индекс уже не существует. Либо можно заполнить данными дисковый индекс, потом перенести их - но в чем тогда разница?
Лучше написать скрипт, который построчно заполнит realtime индекс, и это будет более гибкий инструмент.</li>
</ul>

Предположительно, ATTACH INDEX будет развиваться и в новых версиях Sphinx может работать совсем иначе. Рекомендуется внимательно ознакомиться с документацией перед использованием. Возможно, в вашей компании стоит самая новая версия Sphinx, и вы сможете избежать многих проблем.

<pre>
/www/myapp$ sudo mysql -h 127.0.0.1 --port=9300
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 2.2.10-id64-release (2c212e0)

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]&gt; SHOW tables;
+----------+-------+
| Index    | Type  |
+----------+-------+
| users    | local |
| users_rt | rt    |
+----------+-------+
2 rows in set (0.00 sec)

MySQL [(none)]&gt; DESC users_rt;
+------------+--------+
| Field      | Type   |
+------------+--------+
| id         | bigint |
| first_name | field  |
| last_name  | field  |
| company    | field  |
| login      | string |
+------------+--------+

MySQL [(none)]&gt; ATTACH INDEX users TO RTINDEX users_rt;
Query OK, 0 rows affected (0.00 sec)

MySQL [(none)]&gt; SELECT * FROM users_rt LIMIT 5;
+------+-------------+
| id   | login       |
+------+-------------+
|    1 | aninatalie  |
|    3 | logos       |
|    4 | shumilina   |
|    5 | a.kuzyakova |
|    6 | alkonavt    |
+------+-------------+
5 rows in set (0.03 sec)
</pre>


### INSERT INTO

Основные конструкции для заполнения индекса:

<pre>
INSERT INTO test_rt(id, title, content) VALUES ( 1, 'title1', 'content1')
REPLACE INTO test_rt(id, title, content) VALUES ( 1, 'title2', 'content2')
DELETE FROM test_rt WHERE id = id
</pre>

К сожалению, id - не автоинкрементно, следить за увеличением значения придется самостоятельно. Попытка вставить с помощью INSERT INTO строку с уже существующим id приведет к ошибке.

DELETE FROM можно использовать только с указанием конкретного id. Условие "WHERE id &lt; 1" приведет к ошибке. Попытка вызвать DELETE без WHERE так же приведет к ошибке.
Вот такая унылая ситуация, чтобы удалить пару миллионов записей, надо удалить все эти миллионы поочередно. Надеюсь, в новых версиях ситуация изменилась.

*Почему я ссылаюсь на старые версии Sphinx? Потому что, попробовав установить более современные версии, поняла - такого врагу не пожелаешь. Сразу же нужно обновить модули Sphinx::Search, подобрать те, которые будут совместимы. Тянется куча зависимостей. Из-за обновления Sphinx и Sphinx::Search может перестать работать старый код. Ради добавления нового функционала - переписать тысячи и тысячи строк другого, уже работающего и оттестированного кода - дело не просто дорогое, а очевидно глупое. Поэтому, работаем с тем что есть. А есть у нас - не самая последняя версия Sphinx.*

## Использование realtime индекса в Catalyst-приложении

Изменяем код контроллера. Заменяем Sphinx::Search на DBI, и добавляем код для вставки в realtime индекс новых записей.

```perl
package myapp::Controller::Users;
use Moose;
use namespace::autoclean;
use Data::Dumper;
use DBI;

BEGIN { extends 'Catalyst::Controller'; }

sub index :Path :Args(0) {
    my ( $self, $c ) = @_;

    my $dbh = DBI->connect(
        'DBI:mysql:;host=127.0.0.1;port=9300',
    );

    my $data = $dbh->selectall_arrayref(
        "SELECT id, login FROM users_rt LIMIT 20"
    );

    $dbh->disconnect();

    $c->stash->{config} = Dumper($data);
    $c->stash->{template} = 'users.tt';

    $c->forward('View::TT');
}

sub update :Path('update') :Args(0) {
    my ( $self, $c ) = @_;

    my $dbh = DBI->connect(
        'DBI:mysql:;host=127.0.0.1;port=9300',
    );

    $dbh->do("INSERT INTO users_rt (id, login, first_name, last_name) VALUES 
        (12, 'baby', 'Anna', 'Chu')" );

    my $data = $dbh->selectall_arrayref(
        "SELECT id, login FROM users_rt LIMIT 20"
    );

    $dbh->disconnect();

    $c->stash->{config} = Dumper($data);
    $c->stash->{template} = 'users.tt';

    $c->forward('View::TT');
}

__PACKAGE__->meta->make_immutable;

1;
```

Теперь запускаем сервер.
<pre>/www/myapp$ perl script/myapp_server.pl</pre>

Вводим в адресную строку браузера: http://192.168.33.10:3000/users . Получаем данные из индекса:
<pre>$VAR1 = [ [ '1', 'aninatalie' ], [ '3', 'logos' ], [ '4', 'shumilina' ],
[ '5', 'a.kuzyakova' ], [ '6', 'alkonavt' ] ];
</pre>

Если обратиться по адресу http://192.168.33.10:3000/users/update - в индекс будут добавлены новые данные и тут же получен результат:
<pre>$VAR1 = [ [ '1', 'aninatalie' ], [ '3', 'logos' ], [ '4', 'shumilina' ],
[ '5', 'a.kuzyakova' ], [ '6', 'alkonavt' ], [ '12', 'baby' ] ];
</pre>

## С какими проблемами я столкнулась на данном этапе

### Более строгое поведение строковых атрибутов и полей

Тип данных rt_attr_string позволяет просматривать данные, но не разрешает выполнять поиск по этим данным. Тип rt_field позволяет выполнять поиск, но не позволяет просматривать данные. Если вам нужно и то, и другое, придется добавить в индекс и атрибут, и поле.
<pre>MySQL [(none)]&gt; desc users_rt2;
+------------+--------+
| Field      | Type   |
+------------+--------+
| id         | bigint |
| first_name | field  |
| last_name  | field  |
| company    | field  |
| login      | string |
| first_name | string |
| last_name  | string |
| company    | string |
+------------+--------+
8 rows in set (0.00 sec)

MySQL [(none)]&gt; select * from users_rt2 limit 1;
+-----+------------+------------+-----------+---------+
| id  | login      | first_name | last_name | company |
+-----+------------+------------+-----------+---------+
|   1 | aninatalie | Natalie    | Ani       | dev-lab |
+-----+------------+------------+-----------+---------+
1 row in set (0.01 sec)
</pre>

### Индексы без полнотекстовых полей запрещены

Т.е. если вы хотите просто просматривать данные, без поиска, то создать индекс без полнотекстовых полей не получится:
<pre>index users_rt {
    type=rt
    rt_attr_string = login
    rt_attr_string = first_name
    rt_attr_string = last_name
    rt_attr_string = company

...
}
</pre>

Можно создать пустое поле, не заполнять его данными.

### Альтернатива count() в Sphinx

Для того, чтобы в web-интерфейсе отображать данные постранично, нужно предварительно выяснить общее количество записей в таблице, или количество записей, которые будут найдены с учетом некоторого условия. В обычном MySQL для этого используется функция count().

В Sphinx используется "SHOW META", которая выполняется сразу после основной выборки:
<pre>SELECT * FROM some_index LIMIT 0;

SHOW META;
</pre>
Попытка использовать count() приведет к ошибке:
<pre>MySQL [(none)]&gt; select count(id) from users_rt where match('@first_name anna');
ERROR 1064 (42000): sphinxql: syntax error, unexpected ID,
expecting DISTINCT or '*' near 'id) from users_rt where match('@first_name anna')'
</pre>
Пример:
<pre>MySQL [(none)]&gt; select * from users_rt where match('@first_name anna');
MySQL [(none)]&gt; show meta;
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| total         | 154   |
| total_found   | 154   |
| time          | 0.030 |
| keyword[0]    | anna  |
| docs[0]       | 42514 |
| hits[0]       | 42572 |
+---------------+-------+
6 rows in set (0.01 sec)
</pre>

### Проблема с поиском в большом индексе "offset out of bounds"

Если ваш индекс очень велик, содержит несколько сотен тысяч записей, то могут возникнуть проблемы, когда вы попробуете найти и вывести данные, начиная, к примеру, с 90000-ной записи.
<pre>MySQL [(none)]&gt; SELECT * FROM users_rt LIMIT 90000, 50;
ERROR 1064 (42000): query 0 error: offset out of bounds 
(offset=90000, max_matches=1000)
</pre>
Для решения проблемы можно использовать опцию "max_matches" в запросах, и внести правки в конфиг.

Меняем значение "max_matches" в конфиге, в секции описания демона Sphinx:
<pre>searchd
{
    listen = 127.0.0.1:9300:mysql41
    max_matches = 5000000
...
}
</pre>

При запросе используем "OPTION max_matches":
<pre>SELECT * FROM users_rt LIMIT 90000, 50 OPTION max_matches=500000;</pre>
