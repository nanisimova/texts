# Краткая документация по Apache::Registry

## Что такое Apache::Registry

<font style="color: #990000;">Apache::Registry - это модуль Apache</font>, который позволяет сохранять скомпилированный mod_perl код в памяти для дальнейшего использования.

Дочерний процесс Apache, компилирует код один раз и сохраняет в памяти, где он хранится до его перезагрузки (если скрипт был изменен) или завершения работы процесса. Каждый процесс использует СВОЮ копию скрипта.

При дальнейшем обращении к скрипту, он извлекается из кеша и сразу выполняется. <font style="color: #990000;">Apache::Registry</font> сохраняет в памяти время последней модификации perl-скрипта. Если исходный код был обновлен, скрипт будет скомпилирован заново.

Настройка <font style="color: #990000;">Apache::Registry</font> в httpd.conf :
<pre>
&lt; Directory   /home/aninatalie/aninatalie.ru/cgi&gt;
    SetHandler perl-script
    PerlHandler Apache::Registry
    Options ExecCGI
    ...
&lt; /Directory&gt;
</pre>

Необходимо отметить, что perl-скрипт будет скомпилирован как подпрограмма, входящая в состав другой, внешней, подпрограммы, и получившийся код будет сильно отличаться от исходного. Данную особенность необходимо учитывать при написании кода, который будет работать под управлением <font style="color: #990000;">Apache::Registry</font>. Подпрограммы, изменение которых не желательно, следует помещать в модули. Код модулей при их компиляции и выполнении не оборачивается во внешнюю подпрограмму.

Еще одна особенность работы с <font style="color: #990000;">Apache::Registry</font>: глобальные переменные скрипта сохраняют свои значения между запросами.

<b><font style="color: #990000;">Классический пример того, как будет выглядеть исходный код скрипта после обработки Apache::Registry.</font></b>

Исходный скрипт (<font style="color: #990000;">/cgi/test.pl</font>):

```html
print "Content-type: text/html\n\n";
print "It works\n";
```

Что в реальности будет запускаться на выполнение:

```perl
package Apache::ROOT::cgi::test_e2pl;
use Apache qw(exit);

sub handler {
print "Content-type: text/html\n\n";
print "It works\n";
}
```

Исходный perl-скрипт на выходе выполняется как самостоятельный модуль. Стоит обратить внимание на название пакета: <font style="color: #990000;">Apache::ROOT::cgi::test_e2pl</font>.

Каждый дочерний процесс Apache сохраняет для себя в памяти "готовые к употреблению" скрипты. Один и тот же скрипт может быть помещен в память несколько раз - для каждого процесса отдельно. Во избежание конфликта имен, <font style="color: #990000;">Apache::Registry</font> назначает каждому пакету уникальное название, с помощью добавления уникального ключа к имени пакета.


## Особенности разработки под Apache::Registry

### Использование exit()

Стандартная функция <font style="color: #990000;">Perl - exit() / CORE::exit()</font> - не может использоваться в разработке под mod_perl. Функция приводит к завершению процесса mod_perl. При этом, будут пропущены этапы логирования и корректного завершения работы процесса. Например, не оборвется соединение с базой данных, созданные хендлеры останутся в памяти. Минимальные возможные последствия: утечки памяти.

<font style="color: #990000;">Apache::Registry и Apache::PerlRun</font> негласно замещают стандартную функцию <font style="color: #990000;">exit()</font> функцией <font style="color: #990000;">Apache::exit()</font>. Поэтому, код с <font style="color: #990000;">exit()</font>, который работает под управлением данных модулей, можно использовать без изменений.

Если необходимо принудительно завершить дочерний процесс - рекомендуется использовать
<font style="color: #990000;">Apache::exit (Apache::Constants::DONE)</font>. <font style="color: #990000;">Apache::exit</font> позволит корректно завершить процесс, произведя необходимые записи в логи и соблюдая требования протокола http.

Если необходимо завершить дочерний процесс после того, как завершено выполнение запроса - можно
использовать <font style="color: #990000;">$r-&gt; child_terminate</font> . Данный метод устанавливает значение для переменной <font style="color: #990000;">MaxRequestsPerChild</font> равным 1 и присваивает флагу <font style="color: #990000;">keepalive</font> значение false. После завершения выполнения запроса, родительский процесс инициирует завершение дочернего процесса, если значение <font style="color: #990000;">MaxRequestsPerChild</font> меньше или равно числу выполненных запросов.

### Использование die()

<font style="color: #990000;">die() / CORE::die()</font> - обычно используется для прерывания выполнения программы в случаях, когда что-то идет не так, как ожидалось.

```perl
my $dbh = DBI->connect( @{ $DB_CONNECT_PARAMS },{RaiseError=>0}) or die $DBI::errstr;
```

При срабатывании <font style="color: #990000;">die()</font>, выполнение кода прерывается, печатается причина прерывания и завершается работа интерпретатора Perl.

При работе с mod_perl, завершение работы процесса не желательно. При вызове <font style="color: #990000;">die()</font>, mod_perl делает запись о возникшей ошибке в лог, и вызывает <font style="color: #990000;">Apache::exit()</font> вместо <font style="color: #990000;">CORE::die()</font>.


### Использование STDIN, STDOUT и STDERR</h3>
Под mod_perl, <font style="color: #990000;">STDIN и STDOUT</font> привязаны к сокету, из которого принят запрос.

<font style="color: #990000;">STDERR</font> привязан к файлу, определенному директивой ErrorLog.

Указанные положения исключают возможные конфликты доступа и разделения <font style="color: #990000;">STDIN, STDOUT и STDERR</font>.


### Использование print()

Под mod_perl, <font style="color: #990000;">CORE::print()</font> автоматически переадресует переданные ему аргументы <font style="color: #990000;">Apache::print()</font>.

В контексте mod_perl, нижеприведенные надписи эквивалентны:

```perl
print "Hello";
$r->print("Hello");
```

<font style="color: #990000;">Apache::print()</font> прерывает выполнение и ничего не выводит на печать, если <font style="color: #990000;">$r-&gt;connection-&gt;aborted</font> возвращает true. Это происходит в случае прерывания связи клиентом (например, закрытием браузера).


### Использование __END__ и __DATA__

Скрипты под <font style="color: #990000;">Apache::Registry</font> не могут содержать лексемы <font style="color: #990000;">__END__</font> и <font style="color: #990000;">__DATA__</font> , потому что, во время обработки, исходный код скрипта помещается в код подпрограммы handler().

Если скрипт содержит <font style="color: #990000;">__END__ или __DATA__</font> :

```perl
print "Content-type: text/plain\n\n";
print "Hi";
__END__
Some text that wouldn't be normally executed
```

то, после обработки и перед запуском, выполняемый код будет иметь вид:

```perl
package Apache::ROOT::cgi::test_2epl;
use Apache qw(exit);
sub handler {
    print "Content-type: text/plain\n\n";
    print "Hi";
    _ _END_ _
    Some text that wouldn't be normally executed
}
```

Поскольку Perl игнорирует все, что идет за лексемой <font style="color: #990000;">__END__</font> - программа будет выдавать ошибку.

error.log:
<pre>
[error] Missing right curly or square bracket at /cgi/test.pl line 58, at end of line\nsyntax error 
at /cgi/test.pl line 58, at EOF\n
</pre>


### Переменные окружения $ENV {SERVER_NAME}, $ENV {REMOTE_USER} и др.

<font style="color: #990000;">Apache::Registry</font> эмулирует переменные окружения mod_cgi - <font style="color: #990000;">$ENV {SERVER_NAME}, $ENV {REMOTE_USER}</font> и др.

Отключить данную функциональность можно с помощью конфигурационной директивы PerlSetupEnv Off


### Использование массива @INC

Значение массива @INC при работе с mod_perl, можно изменить только один раз - при запуске сервера. После выполнения каждого запроса к серверу, mod_perl возвращает @INC первоначальное значение.

Если mod_perl встретит в скрипте директиву

```perl
use lib qw(foo/bar);
```

он внесет изменения в @INC, но действовать они будут только до завершения периода компиляции. После этого @INC примет исходное значение.

Есть два способа изменить @INC при запуске сервера:
<ul>
<li>использовать в конфигурационном файле сервера директивы
<pre>PerlSetEnv PERL5LIB /home/httpd/perl</pre>
или
<pre>PerlSetEnv PERL5LIB /home/httpd/perl:/home/httpd/mymodules</pre>
</li>
<li>прописать в startup.pl файле команду:
<pre>use lib qw(/home/httpd/perl /home/httpd/mymodules);
1;
</pre>
и внести изменения в конфигурационный файл сервера, добавив:
<pre>PerlRequire /path/to/startup.pl</pre>
</li>
</ul>


### Перезагрузка модулей

По-умолчанию, сервер не отслеживает факт модификации модулей, подключенных через <font style="color: #990000;">use()</font> или <font style="color: #990000;">require()</font> и считает, будто никаких изменений не производилось. При этом, использует ранее сохраненный в памяти вариант модуля.

На наличие изменений проверяются только perl-скрипты.

Есть несколько способов заставить сервер с mod_perl отслеживать изменения и, при необходимости, перезагружать модули.


#### Перезагрузка сервера

Простейший вариант - просто перезагружать сервер всякий раз, когда вы вносите изменения в Ваш код.

#### Использование Apache::StatINC

После подключения файла, с помощью <font style="color: #990000;">require()</font>, в глобальный хэш <font style="color: #990000;">%INC</font> вносится новая запись. Ключом записи является имя подключенного файла, значением - путь к нему. <font style="color: #990000;">Apache::StatINC</font> просматривает <font style="color: #990000;">%INC</font> и немедленно перезагружает все измененный модули.

Подключить <font style="color: #990000;">Apache::StatINC</font> можно с помощью добавления директив в конфигурационный файл сервера - httpd.conf:
<pre>
PerlModule Apache::StatINC
PerlInitHandler Apache::StatINC
</pre>

*Примечание:*

Если подключается модуль, расположенный в той же директории, что и исходный скрипт, но текущая директория не указана в массиве <font style="color: #990000;">@INC</font>, в <font style="color: #990000;">%INC</font> будет внесена запись типа:

<pre>'MyModule.pm' =&gt; 'MyModule.pm'</pre>

Когда <font style="color: #990000;">Apache::StatINC</font> попытается найти указанный модуль - произойдет ошибка, поскольку он ищет только в директориях, которые указаны в <font style="color: #990000;">@INC</font>. <b>Решение:</b> заранее указывать все необходимые директории в <font style="color: #990000;">@INC</font> .


