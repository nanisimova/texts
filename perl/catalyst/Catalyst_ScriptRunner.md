Catalyst::ScriptRunner

```perl
use Catalyst::ScriptRunner;
Catalyst::ScriptRunner-&gt;run('MyApp', 'Create');
```

Модуль отвечает за загрузку и запуск скриптов в пространстве имен приложения (например, MyApp::Script::Server), или в пространстве имен Catalyst (например, Catalyst::Script::Server).

## Методы

**find_script_class($appname, $script_name)**

На основании переданных данных, ищет и пытается загрузить класс для скрипта. Сначала он попытается найти и загрузить класс "$appname::Script::$script_name" , если попытка окончится неудачно, попробует загрузить "Catalyst::Script::$script_name". Возвращает имя класса.

Т.е. если вызов метода выглядит так:

```perl
use Catalyst::ScriptRunner;

my $class_name = Catalyst::ScriptRunner->find_script_class('MyApp', 'Server');
print $class_name;
```

то будет выполняться поиск - сначала MyApp::Script::Server, затем Catalyst::Script::Server.

<pre>
perl script/myapp_my.pl
Catalyst::Script::Server
</pre>

**run ($application_class, $scriptclass)**

Метод принимает на вход два параметра:
<ul>
<li>класс приложения (например, MyApp);</li>
<li>класс скрипта (чаще всего один из Server/FastCGI/CGI/Create/Test, или др.).</li>
</ul>

Используя find_script_class(), метод run() подгружает найденный класс, создает новый объект загруженного класса и вызывает метод run() уже для него.

Т.е. в каждом модуле Catalyst::Script::&#9913; или MyApp::Script::&#9913; должен быть реализован метод run() , либо унаследован от какого-нибудь базового класса. Например, Catalyst::Script::CGI "наследует" метод run из Catalyst::ScriptRole.

