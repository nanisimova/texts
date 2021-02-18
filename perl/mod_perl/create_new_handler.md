# Как написать свой собственный обработчик (handler) Apache, используя mod_perl? Краткий обзор

Иногда возникает необходимость добавить Apache специфичную функциональность, и одним из вариантов решения данного вопроса является создание нового обработчика.

Подключенный к серверу, mod_perl предоставляет Perl-интерфейс к API Apache. Благодаря Perl-интерфейсу API Apache можно получить доступ к внутренней деятельности Apache из perl-скриптов.

Физически Apache Perl API представляет собой модуль, который работает под управлением mod_perl
- <font color="#009900">Apache.pm</font>. <font color="#009900">Apache.pm</font> и еще несколько модулей, включены в пакет поставки mod_perl.

Apache Perl API позволяет создавать обработчики Apache на Perl для любой фазы обработки запроса.


## Создание обработчика

Внутренняя структура обработчика на perl представляет собой обычный модуль, со специальным методом внутри - <font color="#009900">sub handler</font>.

Пример кода обработчика:

```perl
package Apache::MyHandler;

use strict;
use Apache;
use Apache::Constants qw(:common OK);

sub handler {
	my $r = shift;
	...
	return OK;
}
1;
```

Если метод обработчика имеет отличное от <font color="#009900">handler()</font> - название, необходимо сделать соответствующие дополнения при конфигурировании сервера.

Например, **PerlHandler Apache::MyHandler::not_standart_handler_name** вместо **PerlHandler Apache::MyHandler**.

Для реализации взаимодействия с Apache подключаем основной модуль - <font color="#009900">Apache.pm</font>, и дополнительные
(<font color="#009900">Apache::Constants, Apache::Connection, Apache::Server</font> и т.д.)  - по необходимости.

Согласно неофициальным стандартам, модуль создаваемого обработчика помещается в пространство имен
Apache:

```perl
package Apache::MyModule
```

При вызове обработчика Apache передает ему информацию о текущей транзакции и конфигурации сервера, в виде ссылки на объект. Обработчик может манипулировать этими данными по своему усмотрению.
Согласно сложившимся традициям, ссылка на объект передается переменной, которую называют <font color="#009900">$r</font>.

После завершения работы, метод <font color="#009900">handler</font> должен вернуть серверу определенный статусный код. Для удобства работы с кодами можно использовать модуль <font color="#009900">Apache::Constants</font>.

## Запуск обработчика

Для задания perl-обработчика определенному этапу обработки запроса, в конфигурационном файле сервера (<font color="#009900">httpd.conf</font>) используются директивы <font color="#009900">Perl*Handler</font>. Название каждой директивы позволяет однозначно определить, для какой стадии обработки запроса назначен обработчик.

Директивы <font color="#009900">Perl*Handler</font> предоставляются mod_perl, и не доступны на сервере, где mod_perl не установлен.

Список допустимых директив <font color="#009900">Perl*Handler</font>:
<ul>
<li>PerlChildInitHandler</li>
<li>PerlPostReadRequestHandler</li>
<li>PerlInitHandler</li>
<li>PerlTransHandler</li>
<li>PerlHeaderParserHandler</li>
<li>PerlAccessHandler</li>
<li>PerlAuthenHandler</li>
<li>PerlAuthzHandler</li>
<li>PerlTypeHandler</li>
<li>PerlFixupHandler</li>
<li>PerlHandler</li>
<li>PerlLogHandler</li>
<li>PerlCleanupHandler</li>
<li>PerlChildExitHandler</li>
</ul>

**Пример подключения обработчиков:**
<pre>
&lt; Directory /home/aninatalie/aninatalie.ru/cgi&gt;
	SetHandler perl-script
	PerlInitHandler Apache::StatINC
	PerlHandler Apache::Registry
...
&lt; / Directory&gt;
</pre>
Обработчик <font color="#009900">perl-script</font>, указанный директивой <font color="#009900">SetHandler</font>, означает, что запросы к файлам каталога <font color="#009900">/home/aninatalie/aninatalie.ru/cgi</font> будут обрабатываться с помощью mod_perl. Директива <font color="#009900">PerlInitHandler</font> назначает обработчик <font color="#009900">Apache::StatINC</font> для стадии запроса "Header parsing". Для стадии генерации содержимого и ответа клиенту, директивой <font color="#009900">PerlHandler</font> назначается обработчик <font color="#009900">Apache::Registry</font>.

Стоит отметить, что директивы <font color="#009900">Perl*Handler</font> не обеспечивают автоматическую загрузку модуля обработчика. Для загрузки модуля необходимо предварительно использовать директиву <font color="#009900">PerlModule</font>.

