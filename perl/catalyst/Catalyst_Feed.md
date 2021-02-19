# Создание новостных лент (RSS, Atom) в Catalyst. Создание простого RSS-агрегатора

*Как реализовать собственную новостную ленту в RSS и Atom форматах, на основе Catalyst. Как создать собственный RSS-агрегатор. Простые примеры кода, без комментариев. Работа с модулями XML::Feed и XML::Feed::Aggregator.*

## Реализация собственной RSS-ленты

RSS и Atom — семейство XML-форматов, предназначенных для описания лент новостей, анонсов статей, изменений в блогах и т.п. У каждого формата есть свои преимущества.

Ниже приведен пример реализации собственной новостной ленты, в RSS и Atom форматах.

*/lib/app/Controller/Feed.pm*

```perl
package app::Controller::Feed;
use strict;
use warnings;
use base 'Catalyst::Controller';

use XML::Feed;
use DateTime;
use Encode;

use utf8;

sub atom :Path('/atom') :Args(0) {
  my ($self, $c) = @_;
  $c->stash->{type} = 'Atom';
}

sub rss :Path('/rss') :Args(0) {
  my ($self, $c) = @_;
  $c->stash->{type} = 'RSS';
}

sub end :Private {
  my ($self, $c) = @_;

  my $feed_properties = {
    title => "Test Catalyst Feed",
    link => "http://localhost:3000/",
    description => "Test RSS and Atom and XML::Feed with Catalyst",
    author => "Natalie",
    lang => 'ru-RU',
  };

  my @posts = ({
    title => "Как оптимизировать содержимое таблицы wp_commentmeta в WordPress",
    body => "При создании очередной резервной копии БД, я заметила, что файл с 
      данными стал просто огромным. Детальное изучение показало, что большую часть 
      файла занимают данные из таблицы wp_commentmeta.",
  },{
    title => "Создание системы сессий, авторизации и аутентификации в Catalyst",
    body => "Аутентификация и авторизация пользователей в Catalyst-приложении. 
      Аутентификация на основе DBI. Работа с 
      Catalyst::Authentication::Store::DBI::ButMaintained 
      и Catalyst::Authentication::Credential::Password. ",
  });

  my $feed = XML::Feed->new($c->stash->{type});

  $feed->title($feed_properties->{title});
  $feed->link($feed_properties->{link});
  $feed->description($feed_properties->{description});
  $feed->author($feed_properties->{author});
  $feed->language($feed_properties->{lang});

  foreach my $post (@posts) {
    my $entry = XML::Feed::Entry->new($c->stash->{type});
    $entry->title($post->{title});

    $entry->content($post->{body});
    $entry->issued(DateTime->now);
    $entry->modified(DateTime->now);

    # entry unique ID
    $entry->link($c->req->base.$post->{title});

    $feed->add_entry($entry);
  }

  $c->res->content_type('application/xml');

  if ($c->stash->{type} eq "RSS"){
    $c->res->body($feed->as_xml);
  } else {
    $c->res->body(decode('utf8',$feed->as_xml));
  }
}

1;
```

В реальном приложении данные для новостной ленты будут извлекаться из БД, но в данном примере данные просто размещены в хеше.

До сих пор мы создавали наше Catalyst-приложение, ориентируясь на работу с utf-кодировкой. Для того, чтобы данные в RSS-ленте выводились не в виде крокозябров, добавляем инструкцию "<font color="#00aa00">use utf8;</font>". Разумеется, сам код тоже должен быть сохранен в UTF. Для RSS-ленты этого будет достаточно.

Генерация ленты в Atom-формате происходит с помощью модуля <font color="#00aa00">XML::Atom::Feed</font>, который не очень дружен с utf. Для того, чтобы сгенерированные им данные, можно было прочитать, всю страничку перед публикацией обрабатываем с помощью <font color="#00aa00">decode</font>: <font color="#00aa00">decode('utf8',$feed-&gt;as_xml)</font> .

Запускаем сервер. Новостная лента в RSS-формате доступна по адресу: <font color="#00aa00">http://localhost:3000/rss</font>

Лента в Atom-формате:
<font color="#00aa00">http://localhost:3000/atom</font>

Пример сгенерированного кода для Atom:

```xml
<!--?xml version="1.0" encoding="UTF-8"?-->
<feed xmlns="http://www.w3.org/2005/Atom" xml:lang="ru-RU">
  <title>Test Catalyst Feed</title>
   	<link rel="alternate" href="http://localhost:3000/" type="text/html">
  <subtitle>Test RSS and Atom and XML::Feed with Catalyst</subtitle>
  <author>
    <name>Natalie</name>
  </author>
  <entry>
    <title>Как оптимизировать содержимое таблицы wp_commentmeta в WordPress</title>
    <content type="xhtml">

<div xmlns="http://www.w3.org/1999/xhtml">При создании очередной
      резервной копии БД, я заметила, что файл с 
      данными стал просто огромным. Детальное изучение показало, что большую часть 
      файла занимают данные из таблицы wp_commentmeta.</div>

    </content>
    <published>2014-12-13T18:10:25Z</published>
    <updated>2014-12-13T18:10:25Z</updated>
     	<link rel="alternate" href="http://localhost:3000/Как оптимизировать содержимое
    таблицы wp_commentmeta в WordPress" type="text/html">
  </entry>
</feed>
```

Пример сгенерированного кода для RSS:

```xml
<!--?xml version="1.0" encoding="UTF-8"?>

<rss version="2.0"
 xmlns:atom="http://www.w3.org/2005/Atom"
 xmlns:blogChannel="http://backend.userland.com/blogChannelModule"
 xmlns:content="http://purl.org/rss/1.0/modules/content/"
 xmlns:dcterms="http://purl.org/dc/terms/"
 xmlns:geo="http://www.w3.org/2003/01/geo/wgs84_pos#"
-->

<channel> <title="">Test Catalyst Feed
 	<link>http://localhost:3000/
<description>Test RSS and Atom and XML::Feed with Catalyst</description>
<language>ru-RU</language>
<webmaster>Natalie</webmaster>

<item>
<title>Как оптимизировать 
      содержимое таблицы wp_commentmeta в WordPress</title>
 	<link>http://localhost:3000/Как оптимизировать 
      содержимое таблицы wp_commentmeta в WordPress
<guid ispermalink="true">http://localhost:3000/Как оптимизировать содержимое
      таблицы wp_commentmeta в WordPress</guid>
<pubdate>Sat, 13 Dec 2014 18:09:13 +0000</pubdate>
<content:encoded>При создании очередной 
      резервной копии БД, я заметила, что файл с данными стал просто
      огромным. Детальное изучение показало, что большую часть 
      файла занимают данные из таблицы wp_commentmeta.</content:encoded>
<dcterms:modified>2014-12-13T18:09:13Z</dcterms:modified>
</item>
</channel>
```

## Реализация агрегатора для RSS-лент

RSS-агрегатор - это приложение для автоматического сбора сообщений из RSS-лент. Агрегатор получает список ссылок на RSS-ленты. Далее, самостоятельно по cron, или по запросу клиента, агрегатор проверяет источники на наличие обновлений, и формирует отчет для пользователя в удобном формате.

Ниже приведен пример реализации очень простого RSS-агрегатора. Задается список адресов. По запросу пользователя, все перечисленные адреса опрашиваются, данные сортируются и выводятся на отдельной страничке. Таким образом, клиенту нет необходимость просматривать все интересующие его блоги и сайты, достаточно загрузить одну страницу и увидеть все имеющиеся обновления.

Большинство соц.сетей работают как раз по принципу агрегатора. Пользователь подписывается на новости друзей и тематических сообществ, и всегда видит ленту отсортированных по дате сообщений.

*/lib/app/Controller/FeedAggregator.pm*

```perl
package app::Controller::FeedAggregator;

use strict;
use warnings;
use base 'Catalyst::Controller';

use XML::Feed::Aggregator;
use utf8;

sub index :Path('/feeds') :Args(0) {
  my ( $self, $c ) = @_;

  my $syndicator = XML::Feed::Aggregator->new(
    sources => [
      "http://kawaiisanda.com/feed/",
      "http://dev-lab.info/feed/",
      "http://isif-life.ru/feed"
    ],
    
  )->fetch->aggregate->deduplicate->sort_by_date;

  my @entries = $syndicator->all_entries();

  $c->stash( entries => \@entries );
  $c->stash->{template} = 'feeds.tt';

  $c->forward('View::TT');
}

1;
```

*/root/src/feeds.tt*

```html
[% FOREACH entry = entries %]
  
<a href="[% entry.link %]">[% entry.title %]</a>
  [% entry.author %] - [% entry.issued %]
  [% entry.description %]
[% END %]
```

После запуска сервера можно обратиться по адресу <font color="#00aa00">http://localhost:3000/feeds</font> и получить список всех последних сообщений с выбранных блогов.

Результат:
<pre>
Если не получится, то 1000$ в месяц лишними не будут!
Александр Борисов - 2014-12-08T05:38:00

30 000 — 80 000 рублей в месяц создавая баннеры — интервью с Сергеем Солянским!
Александр Борисов - 2014-12-10T14:26:31

Что такое прерывание
Natalie - 2014-12-11T02:27:39

Как оптимизировать содержимое таблицы wp_commentmeta в WordPress
Natalie - 2014-12-11T12:47:23
</pre>


