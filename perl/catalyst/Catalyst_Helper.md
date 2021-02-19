# Catalyst::Helper

<font color="#00aa00">Catalyst.pl</font> используют для быстрого создания каркаса catalyst-приложения. Сам скрипт <font color="#00aa00">catalyst.pl</font> очень небольшой, единственное, что он делает - вызывает методы Catalyst::Helper, которые выполняют всю работу.

<pre>catalyst.pl  &lt;myappname&gt;</pre>

Пример вывода catalyst.pl:
<pre>
$ catalyst.pl MyApp
created "MyApp"
created "MyApp/script"
created "MyApp/lib"
created "MyApp/root"
...
</pre>

После запуска <font color="#00aa00">catalyst.pl</font> создается целая иерархия файлов, скриптов, директорий:
<pre>lib                   # Lib directory for your app's Perl modules
    MyApp             # Application main code directory
        Controller    # Directory for Controller modules 
        Model         # Directory for Models
        View          # Directory for Views
    MyApp.pm          # Base application module
root                  # Equiv of htdocs, dir for templates, css, javascript
    favicon.ico
    static            # Directory for static files
        images        # Directory for image files used in welcome screen
script                # Directory for Perl scripts
    myapp_cgi.pl      # To run your app as a cgi (not recommended)
    myapp_create.pl   # To create models, views, controllers
    myapp_fastcgi.pl  # To run app as a fastcgi program
    myapp_server.pl   # The normal development server
    myapp_test.pl     # Test your app from the command line
t                     # Directory for tests
    01app.t           # Test scaffold       
    02pod.t           
    03podcoverage.t
Changes               # Record of application changes
Makefile.PL           # Makefile to build application
myapp.conf            # Application configuration file
myapp.psgi
README                # README file
</pre>

## Скрипты, которые создает catalyst.pl

Кроме множества других файлов и директорий, <font color="#00aa00">catalyst.pl</font> создает несколько скриптов, которые в дальнейшем используются для создания новых компонент, запуска простого web-сервера и т.п.
<ul>
<li><font color="#00aa00">myapp_create.pl</font> - Используется для создания новых компонентов catalyst-приложения: моделей, представлений, контроллеров. Пример:
<pre>perl script/myapp_create.pl controller Admin</pre>
</li>

<li><font color="#00aa00">myapp_server.pl</font> -Тестовый web-сервер сatalyst. Запускается как http-демон. Выводит отладочную информацию либо на экран, либо в файл. Лично мне больше нравится использовать файлы, которые можно просматривать в произвольном направлении с помощью less, тогда как вывод логов на экран терминала позволяет посмотреть только ограниченное  количество последних строк.
<pre>CATALYST_DEBUG=1 DBIC_TRACE=1 perl script/myapp_server.pl -p 5000 -r &amp;&gt; server_log</pre>
</li>
<li><font color="#00aa00">myapp_test.pl</font> -Скрипт для запуска тестов из командной строки.</li>
<li><font color="#00aa00">myapp_cgi.pl</font> - Запускает ваше приложение как CGI.</li>
<li><font color="#00aa00">myapp_fastcgi.pl</font> -Скрипт позволяет запустить catalyst-приложение как приложение fastcgi.</li>
</ul>

"myapp" - в приведенных выше примерах, приставка, которая меняется в зависимости от того, какое вы зададите имя для вашего catalyst-приложения при создании каркаса.

## Helpers

Helpers, "хэлперы", модули-помощники.

Скрипт <font color="#00aa00">myapp_create.pl</font> позволяет создавать такие компоненты приложения как: модели, контроллеры, представления. Разновидностей каждого компонента может быть огромное количество. Например, поиск по фразе "Catalyst::View" возвращает более 100 ссылок на модули. Каждое из представлений в catalyst-приложении выполняет свой вид работы, нуждается в разных настройках и кодах. И это только представления. Естественно, модули могут регулярно обновляться, могут появляться новые View.

<font color="#00aa00">myapp_create.pl</font> при всем желании, не сможет содержать в себе код, который используется для создания 100 различных типов представлений. Именно поэтому, <font color="#00aa00">myapp_create.pl</font> отвечает только за прием входящих параметров и передачу их Catalyst::Helper, который в свою очередь, передает часть работы
соответствующим модулям-помощникам. **Модуль-помощник содержит в себе код, необходимый для создания того или иного представления.** Например, для того, чтобы создать представление TT будет вызваны функции модуля Catalyst::Helper::View::TT.

<pre>perl script/myapp_create.pl view TT TT</pre>

Каждый модуль-помощник может содержать в себе два метода:
<ul>
<li><b>mk_compclass</b> - создает класс для компонента</li>
<li><b>mk_comptest</b> - создает тест для компонента. Если метод отсутствует, тест не будет создан.</li>
</ul>

В самом деле, цепочка "передачи обязанностей" по созданию новой компоненты достаточно длинная. Примерное содержимое скрипта <font color="#00aa00">myapp_create.pl</font> :

```perl
use strict;
use warnings;

use Catalyst::ScriptRunner;
Catalyst::ScriptRunner->run('MyApp', 'Create');

1;
```

Т.е. <font color="#00aa00">myapp_create.pl</font> получает задачу создать новую компоненту, вызывает для этого Catalyst::ScriptRunner, Catalyst::ScriptRunner вызывает
<a href="http://dev-lab.info/2015/10/catalystscriptcreate/">Catalyst::Script::Create</a>, Catalyst::Script::Create вызывает Catalyst::Helper, который выполнив определенную работу, вызывает модуль-помощник для создания конкретной компонеты.
Если вы вызовете <font color="#00aa00">script/myapp_create.pl view TT TT</font>, Catalyst::Helper, когда дело дойдет до него, попытается выполнить методы Catalyst::Helper::View::TT-&gt;mk_compclass и Catalyst::Helper::View::TT-&gt;mk_comptest.

Пространства имен для модулей-помощников:
<ul>
<li>Catalyst::Helper::Model::</li>
<li>Catalyst::Helper::View::</li>
<li>Catalyst::Helper::Controller::</li>
</ul>

## Методы Catalyst::Helper

**mk_compclass** - Метод для создания компонента. Catalyst::Helper создает объект модуля-помощника, после этого вызывает для него метод mk_compclass(). Соответственно, в модуле-помощнике этот метод должен быть определен. Если помощник не содержит метод mk_compclass(), Catalyst::Helper передаст управление своему методу render_file().

**mk_comptest** - Аналогично описанному выше, но предназначено не для создания компоненты Catalyst, а теста для компоненты.

**mk_stuff** - Этот метод будет вызван, если пользователь не укажет тип компоненты: модель, представление или контроллер.

## Внутренние методы Catalyst::Helper

**render_file ($file, $path, $vars, $perms)** - Формирует и создает файл из шаблона в блоке __DATA__, который размещают в конце файла модуля-помощника. Файл создается с помощью Template Toolkit.
<ul>
<li>$file - соответствующая часть __DATA__ секции,</li>
<li>$path - путь к файлу,</li>
<li>$vars - ссылка на хэш, которая требуется для Template Toolkit,</li>
<li>$perms - желаемые права доступа (если не указать, будут установлены значения по умолчанию)</li>
</ul>

**get_file($class, $file)** - Извлекает содержимое для файла из __DATA__ секции. Метод используется внутри render_file(). $class - имя класса из которого будут извлекаться данные.

Пример реального кода модуля-помощника:

```perl
package Catalyst::Helper::View::TT;

use strict;

our $VERSION = '0.44';
$VERSION = eval $VERSION;

sub mk_compclass {
    my ( $self, $helper ) = @_;
    my $file = $helper->{file};
    $helper->render_file( 'compclass', $file );
}
1;

__DATA__

__compclass__
package [% class %];
use Moose;
use namespace::autoclean;

extends 'Catalyst::View::TT';

__PACKAGE__->config(
    TEMPLATE_EXTENSION => '.tt',
    render_die => 1,
);

1;
```

**mk_app** - Создает каркас приложения Catalyst, вызывается скриптом <font color="#00aa00">catalyst.pl</font>:

```perl
my $helper = Catalyst::Helper->new( {
    '.newfiles' => !$force,
    'makefile'  => $makefile,
    'scripts'   => $scripts,
    name => $ARGV[0],
} );

$helper->mk_app( $ARGV[0] );
```

**mk_component($app)** - Создает новые компоненты Catalyst-приложения.

**mk_dir($path)** - Создает директорию.

**mk_file($file, $content)** - Добавляет извлеченный с помощью get_file() данные в указанный файл. Вызывается render_file().

**next_test($test_name)** - Вычисляет имя для следующего пронумерованного тестового файла и возвращает его.

**render_file_contents** - Обрабатывает Template::Toolkit шаблон.

**render_sharedir_file** - Формирует template/image файл из директории общего доступа.

Помимо того, что было сказано выше, модули-помощники являются своеобразным архитектурным решением. В Catalyst-приложениях можно встретить хэлперы, которые не предназначены для создания компонент, но выполняют аналогичные работы.


