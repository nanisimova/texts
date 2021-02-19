# Sphinx и Catalyst

*Как реализовать поиск помощью Sphinx в Catalyst-приложении. Как создать приложение Catalyst, конфиги для Sphinx.*

Приложение создавалось на Ubuntu 14.04, perl 5.18, Sphinx версии 2.2.10, модуль Sphinx::Search 0.29. Sphinx от версии к версии меняет функциональность и принципы работы, иногда очень значительно. Модуль Sphinx::Search тоже. Поэтому, не факт, что приведенный ниже код будет работать и у вас в неизменном виде.

В общем, собираясь углубиться в Sphinx тему, надо быть готовым к проблемам совместимости, версионности и т.п.

## Создаем приложение Catalyst

Я часто начинаю заметку с создания Catalyst-приложения с нуля. Делаю это для наглядности. Подобный подход позволяет показать полный код созданных контроллеров, моделей и представлений, не выдирая из приложения только нужные куски. Кроме того, демонстрируемый код не содержит данных, которые относятся к чему-либо еще, кроме рассматриваемой темы.
Создаем директорию www в корневом каталоге. Заходим в нее и создаем приложение:

<pre>/www$ catalyst.pl myapp
created "myapp"
created "myapp/script"
created "myapp/lib"
created "myapp/root"
created "myapp/root/static"
created "myapp/root/static/images"
created "myapp/t"
created "myapp/lib/myapp"
created "myapp/lib/myapp/Model"
created "myapp/lib/myapp/View"
created "myapp/lib/myapp/Controller"
created "myapp/myapp.conf"
created "myapp/myapp.psgi"
created "myapp/lib/myapp.pm"
created "myapp/lib/myapp/Controller/Root.pm"
created "myapp/README"
created "myapp/Changes"
created "myapp/t/01app.t"
created "myapp/t/02pod.t"
created "myapp/t/03podcoverage.t"
created "myapp/root/static/images/catalyst_logo.png"
created "myapp/root/favicon.ico"
created "myapp/Makefile.PL"
created "myapp/script/myapp_cgi.pl"
created "myapp/script/myapp_fastcgi.pl"
created "myapp/script/myapp_server.pl"
created "myapp/script/myapp_test.pl"
created "myapp/script/myapp_create.pl"
Change to application directory and Run "perl Makefile.PL" to make sure your install is complete
</pre>

Создаем представление:
<pre>/www/myapp$ perl script/myapp_create.pl view TT TT
 exists "/www/myapp/script/../lib/myapp/View"
 exists "/www/myapp/script/../t"
created "/www/myapp/script/../lib/myapp/View/TT.pm"
created "/www/myapp/script/../t/view_TT.t"
</pre>

Вносим правки в файл /www/myapp/lib/myapp/View/TT.pm :

```perl
package myapp::View::TT;
use Moose;
use namespace::autoclean;

extends 'Catalyst::View::TT';

__PACKAGE__->config(
    TEMPLATE_EXTENSION => '.tt',
    render_die => 1,
    CATALYST_VAR => 'c',
    ENCODING     => 'utf-8',
);

1;
```

Указываем настройки для TT в файле /www/myapp/lib/myapp.pm . Обращаем внимание, что по умолчанию уже подключен плагин ConfigLoader . Вот его и будем использовать для работы с конфигами.

```perl
package myapp;
use Moose;
use namespace::autoclean;

use Catalyst::Runtime 5.80;

use Catalyst qw/
    -Debug
    ConfigLoader
    Static::Simple
    Unicode::Encoding
/;

extends 'Catalyst';

our $VERSION = '0.01';

__PACKAGE__->config(
    name => 'myapp',
    disable_component_resolution_regex_fallback => 1,
    enable_catalyst_header => 1,
    'View::TT' => {
        INCLUDE_PATH => [
            __PACKAGE__->path_to( 'templates' ),
        ],
    },
    'Plugin::ConfigLoader' => { file => 'conf/myapp.yaml' },
    encoding => 'utf-8',
);

__PACKAGE__->setup();

1;
```

Внутри /www/myapp создаем директорию /conf, внутри нее директорию /sphinx - чтобы отделить конфиги sphinx от остальных.

Создаем контроллер:
<pre>/www/myapp$ perl script/myapp_create.pl controller Users
 exists "/www/myapp/script/../lib/myapp/Controller"
 exists "/www/myapp/script/../t"
created "/www/myapp/script/../lib/myapp/Controller/Users.pm"
created "/www/myapp/script/../t/controller_Users.t"
</pre>
У нас имеется БД mysql, в которой есть таблица - users (достаточно стандартный пример, т.к. таблица users, в том или ином виде, есть почти в любом приложении, где вводится авторизация):
<pre>CREATE TABLE `users` (
	`user_id` INT(11) NOT NULL AUTO_INCREMENT,
	`login` VARCHAR(255) NOT NULL DEFAULT '',
	`first_name` VARCHAR(255) NOT NULL DEFAULT '',
	`last_name` VARCHAR(255) NOT NULL DEFAULT '',
	`company` VARCHAR(255) NOT NULL DEFAULT ''
	PRIMARY KEY (`user_id`),
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB;
</pre>

## Создание Sphinx индекса

Сначала надо подготовить конфиги сфинкса.

/etc/sphinxsearch/sphinx.conf :
<pre>#!/bin/bash

path="/www/myapp/conf/sphinx"

exec /usr/bin/env cat $path/sources.conf $path/searchd.conf $path/indexer.conf $path/users.conf
</pre>
Обычно, конфиг Sphinx выглядит совсем по-другому. Но в данном случае, выносим все данные по отдельным файлам и размещаем в директориях нашего приложения. Так будет намного удобнее вносить изменения. Кроме того, конфиги сфинкса окажутся под "зонтиком" системы контроля версий (если конечно, разработка приложения ведется с использованием git или других систем).

indexer.conf :
<pre>indexer
{
    mem_limit = 256M
}
</pre>
searchd.conf :
<pre>searchd
{
    listen = 127.0.0.1:9300
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
sources.conf :
<pre>source connection {
    type = mysql
    sql_host = localhost
    sql_user = test
    sql_pass = pass
    sql_db   = test
    sql_port = 3306
    sql_query_pre = SET NAMES utf8
}
</pre>
users.conf :
<pre>source users : connection
{
    sql_query_pre = SET NAMES utf8
    sql_query = \
        SELECT user_id, login, first_name, last_name, company FROM users \

    sql_attr_string = login
    sql_field_string = first_name
    sql_field_string = last_name
    sql_field_string = company
}

index users {
    docinfo = extern
    min_stemming_len = 2
    min_word_len = 2
    min_prefix_len = 2
    html_strip = 1
    charset_table = 0..9, A..Z-&gt;a..z, _, a..z, U+410..U+42F-&gt;U+430..U+44F, U+430..U+44F
    source = users
    path = /var/data/sphinx/users
    blend_chars = .
}
</pre>
Запускаем индексацию:
<pre>/www/myapp/conf/sphinx$ sudo indexer users
Sphinx 2.1.9-id64-release (rel21-r4761)
Copyright (c) 2001-2014, Andrew Aksyonoff
Copyright (c) 2008-2014, Sphinx Technologies Inc (http://sphinxsearch.com)

using config file '/etc/sphinxsearch/sphinx.conf'...
indexing index 'users'...
collected 24 docs, 0.0 MB
sorted 0.0 Mhits, 100.0% done
total 24 docs, 329 bytes
total 0.016 sec, 20065 bytes/sec, 1463.77 docs/sec
total 3 reads, 0.000 sec, 1.3 kb/call avg, 0.0 msec/call avg
total 10 writes, 0.000 sec, 0.9 kb/call avg, 0.0 msec/call avg
</pre>
Наш индекс - самый обычный (plain). Добавлять в него данные нельзя. Можно только проводить регулярно переиндексацию (например, специальным скриптом через cron). Если хочется обновления данных в режиме реального времени, придется заморочиться с real-time индексами.

Теперь хорошо бы проверить, был ли создан индекс на самом деле, или нет. Запустим демона Sphinx и попробуем что-нибудь найти.
<pre>/www/myapp/conf/sphinx$ sudo searchd
</pre>

Пробная попытка найти что-нибудь:
<pre>/www/myapp/conf/sphinx$ sudo search --config /etc/sphinxsearch/sphinx.conf Natalie
Sphinx 2.1.9-id64-release (rel21-r4761)
Copyright (c) 2001-2014, Andrew Aksyonoff
Copyright (c) 2008-2014, Sphinx Technologies Inc (http://sphinxsearch.com)

using config file '/etc/sphinxsearch/sphinx.conf'...
index 'users': query 'Natalie': returned 1 matches of 1 total in 0.054 sec

displaying matches:
1. document=1, weight=1724, login=aninatalie, first_name=Natalie,
last_name=Ani, company=dev-lab
</pre>

## Catalyst и Sphinx

Указываем Catalyst-приложению данные для подключения к Sphinx-серверу.

/www/myapp/conf/myapp.yaml :
<pre>sphinx_search:
    host: '127.0.0.1'
    port: '9300'
    timeout: 3
    max_query_time: 3000
</pre>
Добавляем в контроллере коннект к сфинксу и запрос данных.

/www/myapp/lib/myapp/Controller/Users.pm :

```perl
package myapp::Controller::Users;
use Moose;
use namespace::autoclean;
use Data::Dumper;
use Sphinx::Search;
BEGIN { extends 'Catalyst::Controller'; }

sub index :Path :Args(0) {
    my ( $self, $c ) = @_;

    my $sphinx_config = $c->config->{sphinx_search};
    my $sphinx = Sphinx::Search->new();

    my $sphinx_results = $sphinx->SetServer($sphinx_config->{host}, $sphinx_config->{port})
        ->SetConnectTimeout($sphinx_config->{timeout})
        ->SetMaxQueryTime($sphinx_config->{max_query_time})
        ->SetRankingMode(SPH_RANK_SPH04)
        ->SetLimits(0, 1000, 1000)
        ->Query( "Natalie", 'users' );

    if ( $sphinx->GetLastError ) {
        $c->stash->{text} = $sphinx->GetLastError;
    } else {
        $c->stash->{text} = Dumper($sphinx_results);
    }

    $c->stash->{template} = 'users.tt';
    $c->forward('View::TT');
}

__PACKAGE__->meta->make_immutable;

1;
```

В данном случае используется подключение через API. Современные версии Sphinx позволяют получать данные с помощью mysql-клиента, в приложении можно использовать DBI - это намного удобнее. Второй вариант рассмотрим в следующей заметке, про создание real-time индекса.

Теперь нам нужно добавить простую html-страничку, на которой можно будет отобразить все полученные от Sphinx данные.

Рисуем самый простой шаблон.

/www/myapp/templates/users.tt :
<pre>[% text %]</pre>

Теперь попробуем запустить приложение catalyst и организовать поиск по пользователям.
<pre>/www/myapp$ perl script/myapp_server.pl</pre>

В адресной строке браузера вводим http://192.168.56.10:3000/users . Результат:
<pre>$VAR1 = { 'time' =&gt; '0.004', 'matches' =&gt; [ { 'first_name' =&gt; 'Natalie', 'weight' =&gt; 7724,
'company' =&gt; 'dev-lab', 'login' =&gt; 'aninatalie', 'doc' =&gt; 222848, 'last_name' =&gt; 'Ani' } ],
'total_found' =&gt; 1, 'fields' =&gt; [ 'first_name', 'last_name', 'company' ], 'warning' =&gt; '',
'total' =&gt; 1, 'attrs' =&gt; { 'first_name' =&gt; 7, 'company' =&gt; 7, 'login' =&gt; 7, 'last_name' =&gt; 7 },
'words' =&gt; { 'Natalie' =&gt; { 'docs' =&gt; 1, 'hits' =&gt; 1 } }, 'error' =&gt; '' }; 
</pre>

Остановить Sphinx можно с помощью команды:
<pre>/www/myapp$ sudo searchd --stop</pre>

## Проблемы, возникшие в процессе работы со Sphinx

### Не запускается search из командной строки

<pre>/www/myapp$ sudo search user_name
sudo: search: command not found
</pre>
Увы и ах, в последних версиях Sphinx search больше не существует. Можно использовать mysql-клиента или api. Жаль, было удобно.

### Как запретить автозапуск демона Sphinx

Проблема была в автозапуске демона при старте системы. Причем, демона невозможно было убить, процесс моментально возрождался. Возможно, это удобно, когда речь идет о боевом сервере и требуется, чтобы Sphinx был запущен всегда. Но в процессе разработки и отладки, постоянной пересборке real-time индексов - подобное поведение демона недопустимо.

Решение - снять флаг перезапуска в файле /etc/default/sphinxsearch :
<pre>/etc/init$ cat /etc/default/sphinxsearch

START=yes
</pre>

### Ошибка запуска Sphinx демона

Переходим к мистике. При попытке запуска Sphinx демона возникла ошибка. Ошибка видна только в логах сфинкса. Стартует демон "благополучно" и замечаний не выдает. Просто тихо дохнет сразу после выдачи в STDOUT всех положенных сообщений. Текст ошибки в логах:
<pre>sphinx FATAL: binlog meta file /var/lib/sphinxsearch/data/binlog.meta is v.4, binary is v.5;</pre>

Вероятнее всего (предполагаю), ошибка связана с тем, что выполнялись обновления Sphinx и пошел конфликт версий. Ранее созданные файлы данных оказались с новой версией не совместимы.

Что помогло:
<ol>
<li>Удалить все что находится в директории data. Удалить все файлы индексов.
<pre>rm -rf /var/lib/sphinxsearch/data/*
</pre>
</li>
<li>Заново запустить индексацию.
<pre>indexer --all
</pre>
</li>
<li>Запустить демона sphinx.</li>
</ol>

### Как просматривать данные в индексах Sphinx с помощью mysql-клиента

Сначала вносим правки в конфиг сфинкса searchd.conf :
<pre>searchd
{
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

Перезапускаем демона Sphinx.

Используем стандартный консольный mysql-клиент:
<pre>/www/myapp$ sudo mysql -h 127.0.0.1 --port=9300
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 2.2.10-id64-release (2c212e0)

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]&gt; show databases;

MySQL [(none)]&gt; select * from users limit 2;
+-------+-------------+------------+-----------+---------+
| id    | login       | first_name | last_name | company |
+-------+-------------+------------+-----------+---------+
|     1 | aninatalie  | Natalie    | Ani       | dev-lab |
|  1602 | xxxx        |            |           |         |
+-------+-------------+------------+-----------+---------+
2 rows in set (0.00 sec)
</pre>

### Если вы потеряли своего Sphinx демона

В процессе работы и экспериментов, в какой-то момент я довела ситуацию до того, что не могла  ни остановить демона, ни запустить его стандартными средствами. Все попытки заканчивались сообщениями об ошибке.

Проблема заключалась в том, что демон сфинкса уже был запущен на другом порту. Необходимо было найти и прибить его. Очень удобно смотреть кто какой порт занял, с помощью netstat:
<pre>/www/myapp/conf$ sudo netstat -tpln | grep "tcp"
tcp    0    0 0.0.0.0:80       0.0.0.0:*    LISTEN    808/nginx -g daemon
tcp    0    0 127.0.0.1:9311   0.0.0.0:*    LISTEN    3543/searchd
tcp    0    0 0.0.0.0:21       0.0.0.0:*    LISTEN    703/vsftpd
tcp    0    0 0.0.0.0:22       0.0.0.0:*    LISTEN    963/sshd
tcp    0    0 0.0.0.0:1080     0.0.0.0:*    LISTEN    1437/mailcatcher --
tcp    0    0 127.0.0.1:25     0.0.0.0:*    LISTEN    1437/mailcatcher --
tcp    0    0 0.0.0.0:443      0.0.0.0:*    LISTEN    808/nginx -g daemon
tcp    0    0 0.0.0.0:3306     0.0.0.0:*    LISTEN    1269/mysqld
tcp    0    0 127.0.0.1:6379   0.0.0.0:*    LISTEN    1462/redis-server 1
tcp6   0    0 :::80            :::*         LISTEN    808/nginx -g daemon
tcp6   0    0 :::22            :::*         LISTEN    963/sshd
</pre>
Ну и конечно же, "sudo ps -u root -f" никто не отменял.

