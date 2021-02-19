# Как создать Catalyst-приложение с нуля

<i>Как создать Catalyst-приложение с нуля. Какие модули нужно установить. Как создать контроллер, модель и представление. Основные настройки для работы с Catalyst.</i>

<i>Изначально, статья была написана специально для журнала "Pragmatic Perl" (<a href="http://pragmaticperl.com/issues/09/pragmaticperl-09-использование-htmlformfu-при-работе-с-catalyst.html">см. оригинал</a>). Для блога разделила публикацию на две части.</i>

Все примеры были выполнены специально для публикации, в windows-среде, под <font color="#00aa00">Strawberry Perl</font>. В среде unix развернуть все необходимое и заставить работать должно быть даже проще. Приведенные примеры кода будут работать и в том, и в другом случае - они достаточно простые, чтобы не зависеть от тонкостей использования среды.



Данная публикация стала результатом работы над статьей о взаимодействии <font color="#00aa00">HTML::FormFu</font> и <font color="#00aa00">Catalyst</font>.

Для того, чтобы получить более полное представление о работе HTML::FormFu в Catalyst, я создавала Catalyst-приложение с нуля. Более того, для этого мне даже пришлось установить Catalyst под windows. Можно было пойти более простым путем, и использовать уже готовое рабочее unix-окружение, но тогда оставалась вероятность, что в описании я забуду написать про какой-нибудь важный модуль, или аспект, без внимания к которому подключить HTML::FormFu будет не просто.

На своем блоге я решила разделить статью на две части, т.к. первая может быть полезна не только при создании приложения, использующего HTML::FormFu. Уже сейчас, на основе такого же простого каркаса, я планирую создать небольшое REST API и написать по итогам экспериментов заметку.

## Введение

**Что такое Catalyst**

Catalyst - это фреймворк для создания веб-приложений. Поддерживает концепцию MVC (Model-View-Controller). Catalyst поставляется вместе со своим собственным HTTP-сервером, который можно использовать для разработки и тестирования. В боевых условиях, его используют в связке <font color="#00aa00">nginx + FastCGI</font>, или <font color="#00aa00">Apache + mod_perl</font>. Можно использовать другие сервера, но такие решения встречаются значительно реже.

Несмотря на регулярную критику, остается самым мощным и популярным фреймворком в perl-среде.

## Как установить Catalyst под windows

Запускаем cpan в perl-консоли. Потом вводим первую команду:
<pre>force install Catalyst::Runtime</pre>

На этом этапе случился мой первый fail - я долгое время пыталась установить Catalyst (ну правильно, документацию читают только слабаки). Вот так:
<pre>force install Catalyst</pre>

Cpan что-то долго думал, что-то закачивал, устанавливал, но в итоге все заканчивалось ошибкой.  Устанавливать надо было <font color="#00aa00">Catalyst::Runtime</font>, а не Catalyst!

<font color="#00aa00">force install</font> нужен, чтобы запретить cpan слишком много думать и обращать внимание на результаты тестов. При установке perl-модулей под windows - это необходимо.

Потом устанавливаем еще немного дополнительных модулей, без которых сложно построить минимальное Catalyst-приложение или не будет работать HTML::FormFu.
<pre>
force install Catalyst::Controller::HTML::FormFu
force install Catalyst::Devel
force install DBD::mysql
force install DBI
force install DBIx::Class
force install Catalyst::Model::DBIC::Schema
force install Catalyst::View::TT
force install DBIx::Class::Schema::Loader
</pre>
Для того, чтобы беспроблемно установить <font color="#00aa00">DBIx::Class::Schema::Loader</font> - желательно прописать в переменных окружения путь к вашему локальному mysql-демону ( во время тестирования использовалась локальная mysql-БД ).
<pre>
force install MooseX::MarkAsMethods
force install Catalyst::Plugin::Unicode::Encoding
</pre>

## Как создать каркас Catalyst-приложения под Windows

Создаем директорию, в которой будет расположен проект. Например: "*C:\Documents and Settings\username\www*"

Совет: создавая директорию, следует избегать русскоязычных имен в пути к вашему проекту. В дальнейшем это может привести к возникновению проблем.

Переходим в созданную директорию. В командной строке Strawberry Perl вводим:
<pre>catalyst.pl MyApp</pre>

В результате, в каталоге <font color="#00aa00">www</font> будет создана директория с именем <font color="#00aa00">MyApp</font>, содержащая каркас catalyst-приложения.

### Как создать контроллер

Создаем контроллер <font color="#00aa00">Admin.pm</font>, в дальнейшем он нам пригодится. Для этого в perl-консоли выполняем команду:
<pre>perl script/myapp_create.pl controller Admin</pre>

Остальные контроллеры я создавала вручную.

### Как создать View

В отличие от контроллера, view вручную лучше не создавать. В perl-консоли выполняем команду:
<pre>perl script/myapp_create.pl view TT TT</pre>

Команда создаст файл <font color="#00aa00">TT.pm</font> в директории "/lib/MyApp/View/TT.pm" . Модуль TT.pm требуется отредактировать, добавив настройки:

```perl
package MyApp::View::TT;
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

Далее, открываем файл <font color="#00aa00">MyApp.pm</font> и добавляем еще немного конфигурационных данных для TT в блоке <font color="#00aa00">__PACKAGE__-&gt;config()</font>:

```perl
__PACKAGE__->config(
    'View::TT' => {
        INCLUDE_PATH => [
            __PACKAGE__->path_to( 'root', 'src' ),
        ],
    },
    encoding => 'utf-8',
);
```

Кроме того, в блок <font color="#00aa00">use Catalyst qw/.../;</font> добавляем <font color="#00aa00">Unicode::Encoding</font> . Это делается для того, чтобы Catalyst смог нормально обрабатывать русскоязычные символы в шаблонах.

```perl
use Catalyst qw/
    # ...
    Unicode::Encoding
/;
```

Если <font color="#00aa00">Unicode::Encoding</font> не подключить, в дальнейшем можно будет увидеть в логах ошибку:
<pre>[error] Caught exception in engine "Wide character in syswrite at C:/strawberry/
perl/lib/IO/Handle.pm line 474."
Terminating on signal SIGINT(2)
</pre>

### Настройки для HTML::FormFu

В файле MyApp.pm в блок <font color="#00aa00">__PACKAGE__-&gt;config()</font> добавляем настройки:

```perl
__PACKAGE__->config(
    # ...

    'Controller::HTML::FormFu' => {
        'model_stash' => {
            schema => 'DB'
        },
        constructor => {
            tt_args => {
                ENCODING => 'UTF-8',
            }
        }
    }
);
```

### Как создать модель

В perl-консоли выполняем команду:
<pre>perl script/myapp_create.pl model DB DBIC::Schema MyApp::Schema::DB 
create=static "dbi:mysql:test" "root" ""
</pre>
После этого, в блоке <font color="#00aa00">connect_info</font> файла "lib/MyApp/Model/DB.pm" добавляем параметр <font color="#00aa00">mysql_enable_utf8</font> :

```perl
    connect_info => {
        dsn => 'dbi:mysql:test',
        user => 'root',
        password => '',
        mysql_enable_utf8 => 1
    }
```
Этот параметр позволит корректно отображать данные из таблиц БД. Разумеется, если данные хранятся в UTF-формате.

### Как запустить сервер Catalyst под Windows

В perl-консоли выполняем команду:
<pre>perl script/myapp_server.pl</pre>

Это позволит запустить специальный web-сервер. Для боевого использования он не подходит, для тестирования новых возможностей - вполне. В эту же консоль будет выполняться вывод отладочной инфы сервера.

В браузере вводим адрес:
<pre>http://localhost:3000</pre>

