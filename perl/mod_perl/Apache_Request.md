# Краткое описание модуля Apache::Request

При работе с mod_perl, можно использовать объект <font color="#00aa00">Apache::Request</font>. <font color="#00aa00">Apache::Request</font> позволяет получить доступ к данным клиентского запроса. Содержит методы для парсинга запросов GET и POST, Content-type которых имеет значение <font color="#00aa00">application/x-www-form-urlencoded</font> или <font color="#00aa00">multipart/form-data</font>.

<font color="#00aa00">Apache::Request</font> - это более быстрая и легкая альтернатива использованию модуля CGI.pm. Названия методов <font color="#00aa00">Apache::Request</font> и принципы их использования аналогичны методам CGI.pm .

Для работы Apache::Request использует библиотеку <font color="#00aa00">libapreq</font>.
<font color="#00aa00">libapreq</font> - это библиотека, которая предоставляет возможности управления данными клиентского запроса, посредством Apache API.

Функциональность библиотеки:
<ul>
<li>парсинг данных application/x-www-form-urlencoded</li>
<li>парсинг multipart/form-data</li>
<li>парсинг HTTP cookies</li>
</ul>

Кроме <font color="#00aa00">Apache::Request</font>, библиотека <font color="#00aa00">libapreq</font> используется <font color="#00aa00">Apache::Cookie</font>,
для работы с cookies, как альтернатива модулю CGI::Cookie. libapreq написана на C.

## Методы Apache::Request

### new

<pre>use Apache::Request ();
my $apr = Apache::Request-&gt;new($r);
</pre>
Создает новый объект Apache::Request.

При создании объекта можно задавать дополнительные параметры:
<ul>
<li><font color="#00aa00">POST_MAX</font> - Устанавливает ограничение на размер данных, передаваемых методом POST. В байтах. Если лимит будет превышен, при вызове метода parse() будет возвращена ошибка.</li>
<li><font color="#00aa00">DISABLE_UPLOADS</font> - Запрещает загрузку файлов. Метод parse() вернет ошибку, если в запросе будет предпринята попытка загрузить файл.</li>
<li><font color="#00aa00">TEMP_DIR</font> - Назначает каталог для загрузки файлов.
<pre>my $apr = Apache::Request-&gt;new($r, TEMP_DIR =&gt; "/home/httpd/tmp");
my $upload = $apr-&gt;upload('file');
$upload-&gt;link("/home/user/myfile") || warn "link failed: $!";
</pre>
</li>
<li><font color="#00aa00">HOOK_DATA</font></li>
<li><font color="#00aa00">UPLOAD_HOOK</font></li>
</ul>


### instance

Если, в рамках работы с одним запросом, делается попытка создать два и более объектов
<font color="#00aa00">Apache::Request</font>, через вызов метода <font color="#00aa00">new</font>,
то будет обнаружено, что параметры отправленной формы будет содержать только первый объект.
Второй и последующий объекты этих параметров содержать не будут.

<pre>my $apr = Apache::Request-&gt;instance($r, DISABLE_UPLOADS =&gt; 1);</pre>

Использование метода <font color="#00aa00">instance()</font> решает данную проблему. Метод

<font color="#00aa00">instance()</font> позволяет <font color="#00aa00">Apache::Request</font>
существовать в единственном числе (паттерн Singleton).

Благодаря этой схеме, объект Apache::Request с некоторыми данными создается только один раз.
При попытке повторного создания объекта, возвращается ранее созданный экземпляр.

При вызове <font color="#00aa00">instance()</font> можно применять все дополнительные параметры,
которые доступны для метода new(). Поскольку объект фактически создается один раз, при повторных вызовах instance() использование параметров бессмысленно, они учитываются только при создании объекта.

Не желательно использование метода parse() одновременно с <font color="#00aa00">instance()</font>.
Это может привести к ошибкам.

### parse

Метод <font color="#00aa00">parse</font> обеспечивает парсинг запроса. Обычно, по мере необходимости, данный метод неявно вызывается другими методами, и нет необходимости прописывать parse дополнительно, в коде программы. Однако, его удобно использовать для вывода ошибок:

```perl
my $r = shift;
my $apr = Apache::Request->new($r); 

my $status = $apr->parse; 
unless ($status == OK) { 
        $apr->custom_response($status, $apr->notes("error-notes")); 
        return $status; 
} 
```

### param

Метод <font color="#00aa00">param()</font> используется для получения параметров запроса, или их установки. В отличие от CGI.pm, метод <font color="#00aa00">param() Apache::Request</font> работает очень быстро. По скорости выполнения, метод даже превосходит метод <font color="#00aa00">Apache-&gt;args mod_perl</font>.

Синтаксически, использование метода param от <font color="#00aa00">Apache::Request</font> практически не отличается от метода param модуля CGI.pm .

```perl
my $r = shift;
my $apr = Apache::Request->new($r); 

my $value = $apr->param('foo'); # получение значение параметра
my @params = $apr->param;

$apr->param('foo' => [qw(one two three)]); # установка значения параметра
```

### upload

Возвращает единственный объект <font color="#00aa00">Apache::Upload</font> в скалярном контексте,
или все объекты Apache::Upload в контексте списка.

```perl
my $upload = $apr->upload;
my $fh = $upload->fh;
my $lines = 0; 
while(<$fh>) { 
      ++$lines; 
      ...
}
```

Модуль <font color="#00aa00">Apache::Upload</font> предоставляет методы для работы с загружаемыми файлами.

## Пример использования Apache::Request

```perl
#!/usr/bin/perl -w

use strict;
use Apache;
use Apache::Request;

my $r = Apache->request();
my $apr = Apache::Request->new($r);

my $value = $apr->param('name');

print "Content-type: text/plain\n\n";
print $value;

exit;
```
