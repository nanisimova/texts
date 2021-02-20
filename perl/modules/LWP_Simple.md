# Краткое описание LWP::Simple

**LWP::Simple** - это упрощенный интерфейс к LWP.

**LWP::Simple** удобно использовать для вызова в командной строке и написании простых программ.

Если возможностей, которые предоставляет LWP::Simple, недостаточно, рекомендуется использовать LWP::UserAgent.

## Синтаксис

<pre>
perl -MLWP::Simple -e 'getprint "http://aninatalie.ru/"'
</pre>

```perl
use LWP::Simple;

$content = get("http://aninatalie.ru/");
die "Couldn't get it!" unless defined $content;

if (mirror("http://aninatalie.ru/", "foo") == RC_NOT_MODIFIED) {
	# do_somethind
}

if (is_success(getprint("http://aninatalie.ru/"))) {
	# do_somethind
}
```

## Описание

### get($url)

Функция get() обращается к указанному адресу, получает данные и возвращает их пользователю.

Функция вернет undef, если обращение прошло неуспешно. Аргумент $url может быть указан как простая строка или как ссылка на URI-объект.

```perl
#!/usr/local/bin/perl

use LWP::Simple;

my $content = get("http://aninatalie.ru/") || die "Couldn't get it!";
print $content."\n";
```

### head($url)

Функция получает заголовки документа и возвращает 5 значений: $content_type, $document_length, $modified_time, $expires, $server, или TRUE, если требуется получить только одно значение.

Если запрос выполнить не удалось, возвращает пустой список.

Пример:

```perl
#!/usr/local/bin/perl

use LWP::Simple;

my ($content_type, $document_length, $modified_time, $expires, $server) 

= head("http://aninatalie.ru/?p=3");

print "Content type: ".$content_type."\n";

print "Document length: ".$document_length."\n";

print "Modified time: ".$modified_time."\n";

print "Expires: ".$expires."\n";

print "Server: ".$server."\n";
```

<p>Вывод:</p>
<pre>
%perl lwp.pl

Content type: text/html; charset=UTF-8
Document length:
Modified time:
Expires:
Server: Apache/1.3.37 (Unix) PHP/4.4.9
</pre>

### getprint($url)

Функция запрашивает документ по указанному адресу и выводит его на печать. По-умолчанию вывод
осуществляется в STDOUT. Если запрос выполнить не удается, функция выводит ошибку в STDERR.

Пример:

```perl
#!/usr/local/bin/perl

use LWP::Simple;

getprint("http://aninatalie.ru/");
```

Вывод:

```html
<DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
"http://www.w3.org/TR/html4/loose.dtd">

<html>
<head>
<title>
etc.
```

### getstore($url, $file)

Функция обращается по указанному адресу, получает документ и сохраняет его в заданном файле. Возвращает код ответа HTTP.

```perl
#!/usr/local/bin/perl

use LWP::Simple;

my $code = getstore("http://aninatalie.ru/", "store.txt");
print $code;
```

Вывод:
<pre>
%perl lwp.pl

200
</pre>

Содержание store.txt после выполнения скрипта:

```html
<! DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
"http://www.w3.org/TR/html4/loose.dtd">

<html>
<head>
<title>
   etc.
```

