# Использование HTML::FormFu при работе с Catalyst

*Введение в тему использования <font color="#00aa00">HTML::FormFu</font> под <font color="#00aa00">Catalyst</font>. Очень простые примеры, комментарии. Руководство для начинающих. Как создавать и обрабатывать формы.*

Все примеры были выполнены специально для публикации, в windows-среде, под <font color="#00aa00">Strawberry Perl</font>.

## Создаем интерфейс администратора с помощью HTML::FormFu и Catalyst

Панель администратора - это скрытый от обычных посетителей блок сайта, который позволяет владельцу осуществлять управление контентом сайта и принципами его отображения. Например, настроить список используемых виджетов, добавить новые публикации, разместить баннеры и т.п.

Почти вся админка - это бесконечное число разнообразных форм. Использование менеджера форм тут более чем оправдано.

Поэтому, в качестве простого примера, создадим примитивный интерфейс администратора:

<ul>
<li>главную страницу панели администратора</li>
<li>страницу со списком публикаций на сайте</li>
<li>страницу для редактирования статей (на основе HTML::FormFu)</li>
<li>страницу для создания новой публикации (на основе HTML::FormFu)</li>
</ul>

Кроме того, позволим пользователю удалять статьи.

### Дамп БД
Для начала, нам потребуется база данных. Ниже - дамп БД, которая использовалась для приведенных примеров. Установить БД, если ее у вас нет, лучше еще до начала установки Catalyst с его модулями для создания моделей.

Теперь просто создаем таблицу и следим за кодировками, дабы избежать проблем в будущем. Как современные люди, мы выбираем utf-8.
<pre>
-- --------------------------------------------------------
-- Host:                         127.0.0.1
-- Server version:               5.6.14 - MySQL Community Server (GPL)
-- Server OS:                    Win32
-- HeidiSQL version:             7.0.0.4053
-- Date/time:                    2013-10-07 20:42:30
-- --------------------------------------------------------

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET NAMES utf8 */;
/*!40014 SET FOREIGN_KEY_CHECKS=0 */;

-- Dumping database structure for test
CREATE DATABASE IF NOT EXISTS `test` /*!40100 DEFAULT CHARACTER SET utf8 */;
USE `test`;


-- Dumping structure for table test.articles
CREATE TABLE IF NOT EXISTS `articles` (
  `id` int(10) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `full` text,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

-- Dumping data for table test.articles: ~2 rows (approximately)
/*!40000 ALTER TABLE `articles` DISABLE KEYS */;
INSERT INTO `articles` (`id`, `name`, `full`) VALUES
	(1, 'name', 'text'),
	(2, 'name2', 'текст');
/*!40000 ALTER TABLE `articles` ENABLE KEYS */;
/*!40014 SET FOREIGN_KEY_CHECKS=1 */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
</pre>


### Главная страница панели администратора

Для максимально удобной работы с HTML::FormFu в Catalyst, существует модуль <font color="#00aa00">Catalyst::Controller::HTML::FormFu </font>. Если планируем в создаваемом контроллере использовать менеджер форм, подключаем Catalyst::Controller::HTML::FormFu с помощью extends.

Создать форму можно, либо определив ее параметры в непосредственно в коде, либо - по конфигурационному файлу. Конфигурационный файл предпочтительней. Во-первых, это все-таки элементы html-кода, и уже давно стало хорошим правилом отделять разметку от кода. Во-вторых, при создании панели администратора в боевых условиях, форм будет очень много, и удобнее хранить их отдельно.

Чтобы создать форму, нужно задать в атрибутах Catalyst action - <font color="#00aa00">:FormConfig()</font>. Если не указать путь к форме в атрибуте :FormConfig, система решит, что в качестве имени формы следует использовать имя action. Поиск файла будет осуществляться примерно по такому пути: <font color="#00aa00">root/forms/controller_name/action_name.yml</font>

Наличие атрибута <font color="#00aa00">:FormConfig</font> говорит системе, что требуется создать объект формы и разместить его
в хранилище <font color="#00aa00">Catalyst - $c->stash-&gt;{form}</font> . Чтобы вывести форму на html-странице, достаточно добавить в шаблон инструкцию: <font color="#00aa00">[% form %]</font> . form - это полностью готовый блок html-кода формы.

Проверить, была ли отправлена форма и корректны ли ее данные, можно с помощью метода <font color="#00aa00">$form-&gt;submitted_and_valid</font>.

Другой метод, позволит получить данные из полей формы: <font color="#00aa00">$form-&gt;param_value('field_name')</font> . Кроме того, можно получить значения полей формы с помощью <font color="#00aa00">$c-&gt;req-&gt;param('field_name')</font>.

#### Модуль /lib/MyApp/Controller/Admin.pm

```perl
package MyApp::Controller::Admin;
use Moose;
use namespace::autoclean;

BEGIN { extends 'Catalyst::Controller'; }

sub index :Path :Args(0) {
    my ( $self, $c ) = @_;

        $c->stash->{template} = 'admin/index.tt';
        $c->forward('View::TT');
}

__PACKAGE__->meta->make_immutable;

1;
```

#### Шаблон /root/src/admin/index.tt

Переходим в директорию root. Создаем в ней каталог src. В каталоге <font color="#00aa00">src</font> создаем каталог <font color="#00aa00">admin</font>.

Не забываем, что все шаблоны должны быть в кодировке <font color="#00aa00">UTF-8</font> .

```html
<h1>Панель администратора</h1>
<ul>
<li><a href="[% c.uri_for( 'articles' ).path_query %]">Список публикаций</a></li>
</ul>
```

Для наглядности, tt-шаблоны не содержат никакого html-кода, кроме самого необходимого.

### Работа с публикациями

#### Модуль /lib/MyApp/Controller/Admin/Articles.pm

Один модуль будет отвечать и за отображение списка публикаций, и за создание новых, редактирование старых, и за удаление статей.

```perl
package MyApp::Controller::Admin::Articles;
use Moose;
use namespace::autoclean;

BEGIN { extends qw/
    Catalyst::Controller::HTML::FormFu
/ }

=head2 index

Отображение страницы со списком статей

=cut

sub index :Chained('') :Path :Args(0) {
    my ( $self, $c ) = @_;

    $c->stash->{articles_list} = [$c->model('DB::Article')->search(
        {}
    )->all];

    $c->stash->{template} = 'admin/articles.tt';
    $c->forward('View::TT');
}

=head2 remove

Удаление статьи по ее идентификатору с БД

=cut

sub remove :Chained('') :Path('remove') :Args(1) {
    my ( $self, $c, $id ) = @_;

    $c->model('DB::Article')->find({ id => $id})->delete;
    $c->res->redirect($c->uri_for('/admin/articles'));
}

=head2 update

Данный блок кода отвечает за отображение страницы для редактирования статьи,
и за обновление статьи в БД.

=cut

sub update :Chained('') :Path('update') :Args(1) :FormConfig('article/update.yml') {
    my ( $self, $c, $id ) = @_;

    my $form = $c->stash->{form};

    if ($form->submitted_and_valid) {
        eval {
            my $obj = $c->model('DB::Article')->update_or_create({
                id => $form->param_value('id'),
                name => $form->param_value('name') || undef,
                full => $form->param_value('full') || undef
            });
        };

        $c->warn('Duplicate notice task error') if $@;

    } else {
        my $article = $c->model('DB::Article')->search( {
            id => $id,
        } )->first;

        $c->stash->{form}->default_values({
            'name' => $article->name,
            'full' => $article->full,
            'id' => $article->id
        });
    }

    $c->stash->{template} = 'admin/article/update.tt';
    $c->forward('View::TT');
}

=head2 create

Страница для создания новой статьи. Если пользователь нажал кнопку "Сохранить",
статья отправляется в БД, затем клиента перенаправляют на страницу со списком статей.

=cut

sub create :Chained('') :Path('create') :Args(0) :FormConfig('article/create.yml') {
    my ( $self, $c, $id ) = @_;

    my $form = $c->stash->{form};

    if ($form->submitted_and_valid) {
        eval {
            my $obj = $c->model('DB::Article')->create({
                name => $form->param_value('name') || undef,
                full => $form->param_value('full') || undef
            });
        };

        $c->warn('Duplicate notice task error') if $@;

        $c->res->redirect($c->uri_for('/admin/articles'));
    } else {
    
        $c->stash->{template} = 'admin/article/create.tt';
        $c->forward('View::TT');
    }
}

__PACKAGE__->meta->make_immutable;

1;
```

Метод <font color="#00aa00">default_values()</font> - позволяет задать значения "по-умолчанию" полям формы, то, что увидит пользователь, если откроет страницу с формой.

Можно использльзовать метод <font color="#00aa00">$form-&gt;submitted</font> , чтобы понять - была форма отправлена или нет. Форма может быть отправлена, но не пройти валидацию. Метод <font color="#00aa00">$form-&gt;submitted_and_valid</font> учитывает и отправку, и благополучное прохождение валидации.

#### Шаблон /root/src/admin/articles.tt

```html
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>Формочка</title>
     	<link rel="stylesheet" type="text/css" href="/static/style.css">

[% FOREACH article = articles_list %]
[% do_something %]    
[% END %]

<table id="result_list" cellspacing="0">
<tbody>
<tr>
<td>
        <a href="[% c.uri_for( 'update', article.id ).path_query %]">
            [% article.name %]</a>
        <a href="[% c.uri_for( 'remove', article.id ).path_query %]">Удалить</a>
</td>
</tr>
</tbody>
</table>
<a href="[% c.uri_for( 'create' ).path_query %]">Добавить новую статью</a>
```

#### Шаблон /root/src/admin/article/update.tt

```html
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>Формочка</title>
     	<link rel="stylesheet" type="text/css" href="/static/style.css">
Update article
[% form %]
```

#### Шаблон /root/src/admin/article/create.tt

```html
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>Формочка</title>
     	<link rel="stylesheet" type="text/css" href="/static/style.css">
Create new article
[% form %]
```

#### CSS-стили для формы /root/static/style.css

Добавим совсем простую css-таблицу, которая позволяет аккуратно вывести элементы формы. В дальнейшем, css можно усложнить, создавая с его помощью профессиональное оформление всей панели администратора.

```html
form {
  width: 40em;
}
 
.submit {
  display: block;
}
 
label {
  display: block;
}
```

#### YML-конфиг для формы /root/forms/article/update.yml

По-умолчанию, catalyst-приложение будет искать формы в директории <font color="#00aa00">root/forms</font> . Для работы с конфигурационными файлами, <font color="#00aa00">Catalyst::Controller::HTML::FormFu</font> использует <font color="#00aa00">Config::Any</font>. Соответственно, хранить конфигурационную информацию о формах кроме yml-формата, можно в XML, JSON, конфигах в стиле Apache, perl-коде и т.п.

Переходим в директорию <font color="#00aa00">root</font>. Создаем в ней каталог <font color="#00aa00">forms</font>. В каталоге forms создаем каталог <font color="#00aa00">article</font> и файл <font color="#00aa00">update.yml</font>.

```
---
attributes:
    id: element-form

auto_fieldset:
    attributes:
        class: module aligned

---
elements:
    - type: Hidden
      name: id

    - type: Text
      name: name
      label: Название статьи

    - type: Textarea
      name: full
      label: Текст статьи

    - type: Submit
      name: submit
      value: Сохранить
```

#### YML-конфиг для формы /root/forms/article/create.yml

```
---
attributes:
    id: element-form

auto_fieldset:
    attributes:
        class: module aligned

---
elements:
    - type: Text
      name: name
      label: Название статьи

    - type: Textarea
      name: full
      label: Текст статьи

    - type: Submit
      name: submit
      value: Сохранить
```

Вот и все. Простой прототип панели администратора с использованием HTML::FormFu готов. Теперь, на основе полученных результатов, можно пробовать усложнять формы, вводить в работу сложные поля и правила валидации, добавлять JS для работы со сложными элементами, CSS для создания современного интерфейса.

