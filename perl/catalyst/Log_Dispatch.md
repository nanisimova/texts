# Ведение логов в Catalyst с помощью Log::Dispatch

<font color="#00aa00">Log::Dispatch</font> помогает задать - насколько подробно будет осуществляться логирование. Будут ли выводиться в лог только критические ошибки, или будут включены и отладочные сообщения, или уровень, начиная с не критичных предупреждений. Кроме того, с помощью <font color="#00aa00">Log::Dispatch</font> удобно задавать объекты для вывода сообщений.

## Пример использования Log::Dispatch в Catalyst-приложении

<a href="Catalyst_intro.md" target="_blank">Как создать Catalyst-приложение с нуля</a>

*app::Log.pm* :

```perl
package app::Log;

use uni::perl;
use base 'Log::Dispatch';

sub new {
    my $class = shift;

    my $log = $class->SUPER::new(
        outputs => [
           $ENV{'DEBUG'}.
                ? [ 'Screen', min_level => 'debug', newline => 1, stderr => 1 ]
                : [ 'File', min_level => 'debug', newline => 1, 
                  filename => 'devlogfile' ],
        ],
        @_,
    );
    return $log;
}

1;
```

По умолчанию, для управления логами Catalyst использует <a href="Catalyst_Log.md">Catalyst::Log</a>. Если необходимо вмешаться в процесс логирования, используется инструкция:

```perl
__PACKAGE__->log( MyLogger->new );
__PACKAGE__->setup;
```

и в дальнейшем, команды типа:
<pre>$c->log->info( 'Now logging with my own logger!' );</pre>

*app.pm* :

```perl
use app::Log;

__PACKAGE__->log(app::Log->new());
__PACKAGE__->setup();
```

*Root.pm :*

<pre>$c->log->debug('ARG: '.$c->stash->{domain});</pre>

С помощью переменной <font color="#00aa00">DEBUG</font> реализуется вариативность работы с логами. Если переменная задана:
<pre>DEBUG=1 perl script/app_server.pl</pre>
вывод всей отладочной информации осуществляется прямо на экран рабочего терминала.

Если переменная <font color="#00aa00">DEBUG</font> не задана:
<pre>perl script/app_server.pl</pre>

запись данных будет вестись в файл <font color="#00aa00">devlogfile</font>. Если файла <font color="#00aa00">devlogfile</font> не существует, приложение создаст его.

Пример записи в devlogfile:
<pre>
*** Request 1 (0.200/s) [19227] [Thu Jul 24 22:26:10 2014] ***
Path is "/"
"GET" request for "/" from "190.10.154.66"
ARG: dev-lab.info
</pre>

С помощью дополнительных модулей, можно организовать вывод логов в таблицы базы данных (<font color="#00aa00">Log::Dispatch::DBI</font>), через Jabber (<font color="#00aa00">Log::Dispatch::Jabber</font>), в окно Tk (<font color="#00aa00">Log::Dispatch::Tk</font>), в окно windows event log (<font color="#00aa00">Log::Dispatch::Win32EventLog</font>), и множество других интересных вариантов: <font color="#00aa00">Log::Dispatch::Syslog</font>, <font color="#00aa00">Log::Dispatch::ApacheLog</font>, <font color="#00aa00">Log::Dispatch::Email</font>, и т.п.

Данная вариантивность особенно удобна, при запуске Catalyst-приложения в разных средах, с разным окружением и для разных целей. Ведение логов на боевых серверах отличается от ведения логов в момент разработки.

