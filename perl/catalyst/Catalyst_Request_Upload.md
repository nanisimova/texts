# Как загружать файлы в Catalyst-приложении. Catalyst::Request::Upload

*Загрузка файлов в Catalyst-приложении с помощью стандартного средства Catalyst::Request::Upload. Работа с библиотекой lib::abs.*

Задача: требуется организовать загрузку пользовательских файлов на сайт.

Реализация:
<ol>
<li>Страница с полным списком товаров и вывод пригрепленного к товару изображения.</li>
<li>Страница добавления нового товара.</li>
<li>Страница редактирования товара и поле для загрузки изображений.</li>
</ol>

Пример очень простой, который демонстрирует только общие принципы работы с загружаемыми
пользовательскими файлами в Catalyst-приложении. В приведенном примере предполагалась загрузка только изображений, хотя тот же код можно использовать для загрузки файлов других типов: архивы, документы, и т.п.

## Изменения в базе данных

Создаем новую таблицу в базе данных mysql, которая содержит информацию о товарах. В нашем случае она будет содержать только идентификатор товара, его название и название привязываемого к нему изображения.
<pre>CREATE TABLE `catalog` (
	`id` INT(10) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(255) NULL DEFAULT NULL,
	`img_name` VARCHAR(255) NULL DEFAULT NULL,
	PRIMARY KEY (`id`)
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB;
</pre>

## Модель - доступ к файловому хранилищу

Зачем нам тут модель? Файлы, после получения от клиента - надо куда-то сохранять. Можно сохранять файлы прямо в специально отведенные директории, в рамках файловой структуры сайта.

Мне эта идея не нравится. Представим, что это не маленький интернет-магазин, а магазин с сотнями позиций, и каждую из них иллюстрирует несколько изображений, а к некоторым позициям добавляется видео с демонстрацией использования.

Еще хуже, если реализуется социальная сеть. Не так уж редко, один пользователь загружает больше тысячи фотографий себя любимого, и еще некоторое количество видео-файлов. Умножим это количество файлов на несколько тысяч пользователей (не говоря уж о том, что пользователей может быть несколько сотен тысяч).

Разместить такое количество данных в директории /myapp/root/images - занятие крайне не благодарное и опасное. Лучше использовать для хранения файлов специальные файловые хранилища и отдельные сервера, которые будут специализироваться на работе со статикой.

Вариантов реализации файлового хранилища для сайта, может быть огромное множество, в зависимости от потребностей и возможностей проекта. Но в любом случае, для работы с этим файловым хранилищем нужна модель.

**Модель целиком и полностью отвечает за доступ к файлам, сохранение файлов, их удаление, проверку типов, назначение новых имен.** Остальное приложение не знает, где хранятся файлы, как организован доступ к ним, какой протокол используется для работы с хранилищем и т.п. Если вы вдруг решите изменить принципы работы хранилища, вам понадобится переписать только код модели, а не всего приложения.

В моем примере модель реализуется самая простая, главное просто обозначить, что она есть. Проверки типов, имен файлов - не производятся. Если попытаться записать дважды файл, с одним и тем же именем, вероятнее всего, произойдет ошибка. Файлы сохраняются в отдельную директорию проекта. Как уже упоминалось выше, это не правильно "in real life", но для примера - вполне допустимо.

\lib\app\Model\FileStorage.pm :

```perl
package app::Model::FileStorage;

use uni::perl ':dumper';
use Moose;

use strict;
use utf8;
use lib::abs;

use Path::Class::File;

sub put {
    my ( $self, $upload ) = @_;

    $upload->copy_to('root/static/filestorage/images/'.$upload->filename);

    return 1;
}

sub get_file_url {
    my ( $self, $filename ) = @_;

    return "/static/filestorage/images/".$filename;
}

sub delete {
    my ( $self, $filename ) = @_;

    my $path = lib::abs::path('../../../root/static/filestorage/images/');
    my $file = Path::Class::File->new($path, $filename);
    my $res = $file->remove();
 
    return $res;
}

1;
```

### Библиотека lib::abs, документация
Библиотека <font color="#00aa00">lib::abs</font> помогает преобразовать относительные пути в абсолютные. Пример:

```perl
use lib::abs;

my $path = lib::abs::path('../dev-lab.info/zvz.txt');
print $path."\n";
```

Результат выполнения:
<pre>
&gt; perl abs.pl
C:/Documents and Settings/administrator/My Documents/dev-lab.info/vzv.txt
</pre>

Допустимый вариант использования:

```perl
use lib::abs qw(./lib ../bin);
use lib::abs 'libs';
```

Указанные инструкции будут обрабатываться в тот же момент, когда начнется стадия выполнения <font color="#00aa00">BEGIN</font>-блока, и повлияет на то, какие данные будут помещены в <a href="http://dev-lab.info/2011/08/что-такое-inc-и-inc">@INC</a>. Но такой вариант применения <font color="#00aa00">lib::abs</font> встречается значительно реже чем первый, с использованием функции <font color="#00aa00">path()</font>.

## Контроллер - загрузка файлов, их удаление и просмотр

Особенности реализации.

Так же, как и модель, контроллер сильно упрощен. Отсутствуют проверки на ошибки, нет возможности добавлять несколько изображений к одному товару. При создании нового элемента, добавление картинки не реализуется.

\lib\app\Controller\Admin\Catalog.pm :

```perl
package app::Controller::Admin::Catalog;
use Moose;
use namespace::autoclean;

use app::Model::FileStorage;

BEGIN { extends 'Catalyst::Controller'; }

# вывод полного списка элементов каталога
sub list :Path('/admin/catalog') :Args(0) {
    my ( $self, $c ) = @_;

    my $dbh = $c->model( 'DBI' )->dbh;
    my $filestorage = app::Model::FileStorage->new;

    my $catalog = $dbh->selectall_arrayref( "select * from catalog order by id", 
        { Slice => {} } );

    foreach my $element (@$catalog) {
        $element->{link} = $filestorage->get_file_url($element->{img_name});
    }

    $c->stash( catalog => $catalog );

    $c->stash->{template} = 'admin/catalog/list.tt';
    $c->forward('View::TT');
}

# добавление нового элемента каталога
sub element_create :Path('/admin/catalog/create') :Args(0) {
    my ( $self, $c ) = @_;

    my $name = $c->request->params->{name} || "";

    if (length($name) > 1) {
        my $dbh = $c->model('DBI')->dbh;
        my $sql = qq{INSERT INTO catalog(name, img_name) VALUES ('$name', '')};
        my $rows = $dbh->do($sql);
        $c->response->redirect($c->uri_for('/admin/catalog'));
    }

    $c->stash->{template} = 'admin/catalog/element_create.tt';
    $c->forward('View::TT');
}

# редактирование элемента каталога, тут же может быть добавлена картинка
sub element_edit :Chained('chain4') :PathPart('edit') :Args(0) {
    my ( $self, $c ) = @_;

    my $id = $c->stash->{id};
    my $name = $c->request->params->{name} || "";

    my $filestorage = app::Model::FileStorage->new;
    my $dbh = $c->model('DBI')->dbh;
    if (length($name) > 1) {
        my $upload = $c->req->upload('img_name');

        my $res = $filestorage->put($upload);
        my $img_name = ($res ? $upload->filename : '');

        my $sql = qq{UPDATE catalog SET name='$name', img_name = '$img_name'
            WHERE id='$id'};

        my $rows = $dbh->do($sql);
        $c->response->redirect($c->uri_for('/admin/catalog'));
    }

    my $element = $dbh->selectrow_hashref(qq{SELECT id, name, img_name 
    FROM catalog WHERE id = '$id'});

    $element->{link} = $filestorage->get_file_url($element->{img_name});
    $c->stash->{element} = $element;

    $c->stash->{template} = 'admin/catalog/element_edit.tt';
    $c->forward('View::TT');
}

# удаление элемента каталога и удаление связанной картинки
sub element_delete :Chained('chain4') :PathPart('delete') :Args(0) {
    my ( $self, $c) = @_;

    my $id = $c->stash->{id};
    my $dbh = $c->model('DBI')->dbh;
    my $filestorage = app::Model::FileStorage->new;

    my $element = $dbh->selectrow_hashref(qq{SELECT id, name, img_name 
    FROM catalog WHERE id = '$id'});
    my $res = $filestorage->delete($element->{img_name});

    my $sql = qq{DELETE FROM catalog WHERE id = '$id'};
    my $rows = $dbh->do($sql);

    $c->response->redirect($c->uri_for('/admin/catalog'));
}

sub chain1 :PathPart('') :Chained('/') :CaptureArgs(0) {
    my ( $self, $c) = @_;
}

sub chain2 :Chained('chain1') :PathPart('admin') :CaptureArgs(0) {
    my ( $self, $c) = @_;
}

sub chain3 :Chained('chain2') :PathPart('catalog') :CaptureArgs(0) {
    my ( $self, $c) = @_;
}

sub chain4 :Chained('chain2') :PathPart('catalog') :CaptureArgs(1) {
    my ( $self, $c, $id ) = @_;

    $c->stash->{id} = $id;
}

__PACKAGE__->meta->make_immutable;

1;
```

В результате, данное приложение после запуска будет реагировать на следующие адресам:
<ul>
<li><font color="#00aa00">http://localhost:3000/admin/catalog</font></li>
<li><font color="#00aa00">http://localhost:3000/admin/catalog/create</font></li>
<li><font color="#00aa00">http://localhost:3000/admin/catalog/1/edit</font></li>
<li><font color="#00aa00">http://localhost:3000/admin/catalog/1/delete</font></li>
</ul>

### Модуль Catalyst::Request::Upload, документация

<font color="#00aa00">Catalyst::Request::Upload</font> - обрабатывает запросы на загрузку файлов в Catalyst. Чтобы задать место хранения временных файлов Catalyst, надо указать в конфиге:
<pre>__PACKAGE__->config( uploadtmp => '/path/to/tmpdir' );
</pre>
По умолчанию, Catalyst будет использовать системную директорию для хранения временных файлов - <font color="#00aa00">temp</font> .

**$upload-&gt;new**

Создание нового объекта <font color="#00aa00">Catalyst::Request::Upload</font>. Объект создается автоматически, при вызове
<font color="#00aa00">my $upload = $c-&gt;req-&gt;upload('field');</font>

**$upload-&gt;copy_to**

Копирует временный файл в указанную директорию. Копирование реализовано с помощью <font color="#00aa00">File::Copy</font>. Возвращает <font color="#00aa00">true</font> в случае успешного завершения операции, и <font color="#00aa00">false</font> - в случае неудачи.

**$upload-&gt;fh**

Возвращает файловый дескриптор (<font color="#00aa00">IO::File</font>) для временного файла.

**$upload-&gt;link_to**

Создает жесткую ссылку для временного файла. Возвращает <font color="#00aa00">true</font> в случае успешного завершения операции, и <font color="#00aa00">false</font> - в случае неудачи.

**$upload-&gt;headers**

Возвращает объект <font color="#00aa00">HTTP::Headers</font> для запроса, во время которого был загружен файл.

&nbsp;

Общий пример для нескольких методов, приведенных ниже:

```perl
print "INFO: ".$upload->filename." | ".$upload->size." | ".
$upload->basename." | ".$upload->tempname." | ".$upload->type."\n";
```

Вывод (запуск скрипта был под windows):
<pre>
INFO: ny_small.jpg | 47595 | ny_small.jpg | 
C:\DOCUME~1\administrator\LOCALS~1\Temp\Qq6DEOrd_O.jpg | image/jpeg
</pre>

**$upload-&gt;size**

Возвращает размер загруженного файла в байтах.

**$upload-&gt;basename**

Возвращает имя файла. При этом, имя будет обработано регулярным выражением <font color="#00aa00">basename =~ s|[^\w\.-]+|_|g</font> , чтобы избежать появления имен со специальными символами, которые могут некорректно обрабатываться операционной системой.

**$upload-&gt;tempname**

Возвращает путь к временному файлу, который был создан Catalyst-приложением после получения запроса от клиента.

**$upload-&gt;type**

Возвращает <font color="#00aa00">Content-Type</font> для отправленного клиентом файла.


*Для получения более полной информации, смотрите официальную документацию на модуль Catalyst::Request::Upload.*

## Шаблоны - формы для загрузки файлов

**1.** Страница со списком товаров
root\src\admin\catalog\list.tt :

```html
[% FOREACH element = catalog %]
   [% do_something %]
[% END %]

<table>
<tbody>
<tr>
<td>
   <a href="/admin/catalog/[% element.id %]/edit">[% element.name %]</a>
   <a href="/admin/catalog/[% element.id %]/delete">Удалить</a>
   <img src="[% element.link %]">
</td>
</tr>
</tbody>
</table>
<a href="/admin/catalog/create">Добавить новый товар</a>
```

**2.** Страница создания нового товара
root\src\admin\catalog\element_create.tt :

```html
<form action="/admin/catalog/create" method="POST">
Название товара
<input type="text" size="20" name="name" maxlength="250">
Картинка товара
<input type="text" size="20" name="img_name" maxlength="250">
<input value="Создать новую позицию" type="submit">
</form>
```

**3.** Страница редактирования товара
root\src\admin\catalog\element_edit.tt :

```html
<form action="/admin/catalog/[% element.id %]/edit" method="POST" enctype="multipart/form-data">
Название товара
<input type="text" size="20" name="name" maxlength="250" value="[% element.name %]">
Картинка товара
<input type="file" size="20" name="img_name" maxlength="250" value="[% element.img_name %]">
<input value="Обновить данные" type="submit">
</form>
<img src="[% element.link %]">
```

**4.** В каталоге <font color="#00aa00">root</font> должен быть создан каталог <font color="#00aa00">static</font>, в каталоге <font color="#00aa00">static</font> создаем каталог <font color="#00aa00">filestorage</font>, в каталоге <font color="#00aa00">filestorage</font> создаем каталог <font color="#00aa00">images</font>.

**5.** Для того, чтобы картинки отображались, в модуле <font color="#00aa00">\lib\app.pm</font> правим конфигурационный блок:

```perl
__PACKAGE__->config(
    'Plugin::Static::Simple' => {
        include_path => [
            '/static',
            app->config->{root},
        ],
        ignore_dirs => [ qw/images css js help filestorage/ ],
    },
);
```

