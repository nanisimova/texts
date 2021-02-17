# Работа с cookies и CGI::Application

## Установка cookies

<pre>sub on_set_cookies {
	my $self = shift;
	my $query = $self-&gt;query();

	my $auth_cookie = $query-&gt;cookie( # подготавливаем cookie
			-name=&gt;'alisa',
			-value=&gt;'pass',
			-expires=&gt;'+1h',
			-path=&gt;'/cgi',
			-domain=&gt;'.aninatalie.ru',
	);

	$self-&gt;header_props(-cookie=&gt;[$auth_cookie]); # устанавливаем у клиента

	return $self-&gt;tt_process('template2.tt');
}

</pre>

## Получение информации из cookies

<pre>sub on_set_cookies {
	my $self = shift;
	my $query = $self-&gt;query();

	my $auth_cookie = $query-&gt;cookie('alisa');
		# Значение переменной $auth_cookie будет равно 'pass'. 
		# Т.е. мы получим только значение -value .
	...
	return $self-&gt;tt_process('template2.tt');
}
</pre>

## Обновление данных cookies

Чтобы обновить значение <font color="#00aa00">cookie</font>, надо просто записать новые данные поверх существующих. Для этого указываем <font color="#00aa00">-name</font> нужной <font color="#00aa00">cookie</font>, и задаем новые параметры для нее.

## Удаление cookies

Удалить <font color="#00aa00">cookie</font> тоже просто - обновляем значение нужной <font color="#00aa00">сookie</font> и <font color="#00aa00">-expires</font> присваиваем отрицательное значение, например, <font color="#00aa00">-1d</font>. Браузер примет обновление <font color="#00aa00">сookie</font> и, увидев отрицательное время, посчитает, что он давным давно уже должен был удалить <font color="#00aa00">сookie</font> (в данном случае, еще вчера). И удалит.
<pre>sub on_set_cookies {
	my $self = shift;
	my $query = $self-&gt;query();

	my $auth_cookie = $query-&gt;cookie(
			-name=&gt;'alisa',
			-value=&gt;'pass',
			-expires=&gt;'-1h',
			-path=&gt;'/cgi',
			-domain=&gt;'.aninatalie.ru',
	);

	$self-&gt;header_props(-cookie=&gt;[$auth_cookie]);

	return $self-&gt;tt_process('template2.tt');
}
</pre>

## Памятка
### Что такое cookies

<font color="#00aa00">Cookie</font> является решением одной из наследственных проблем HTTP протокола (HyperText Transfer Protocol). Проблема заключается в непостоянстве соединения между клиентом и сервером.

Используя <font color="#00aa00">cookie</font>, можно эмулировать сессию по HTTP протоколу. Коротко принцип эмуляции сессии таков: на первом запросе выдается соотвествующее значение <font color="#00aa00">cookie</font>, а при каждом последующем запросе это значение читается из переменной окружения HTTP_COOKIE и соответствующим образом обрабатывается.

<font color="#00aa00">Cookie</font> - это небольшая порция текстовой информации, которую сервер передает браузеру. Браузер будет хранить эту информацию и передавать ее серверу с каждым запросом как часть HTTP заголовка.

Некоторые <font color="#00aa00">cookie</font> могут храниться только в течение одной сессии, они удаляются после закрытия браузера. Другие, установленные на определенный период времени, записываются в файл(ы).

### Формат и синтаксис cookie

<font color="#00aa00">Сookie является частью HTTP заголовка</font>. Полное описание поля Set-Cookie HTTP заголовка:
<pre>Set-Cookie: name=VALUE; expires=DATE; path=PATH; domain=DOMAIN_NAME; secure</pre>

Минимальное описание поля Set-Cookie HTTP заголовка:
<pre>Set-Cookie: name=VALUE;</pre>

<ul>
<li><b>name=VALUE</b> - строка символов, исключая перевод строки, запятые и пробелы. NAME-имя
cookie, VALUE - значение. Не допускается использование двоеточия, запятой и пробела.</li>

<li><b>expires=DATE</b> - время хранения cookie, т.е. вместо DATE должна стоять дата в формате
"expires=Monday, DD-Mon-YYYY HH:MM:SS GMT", после которой истекает время хранения cookie. Если
этот атрибут не указан, то cookie хранится в течение одного сеанса, до закрытия броузера.</li>

<li><b>domain=DOMAIN_NAME</b> - домен, для которого значение cookie действительно. Например,
"domain=cit-forum.com". Если этот атрибут опущен, то по умолчанию используется доменное имя сервера, на котором было
задано значение cookie.</li>

<li><b>path=PATH</b> - атрибут устанавливает подмножество документов, для которых действительно
значение cookie. Для того, чтобы cookie отсылались
при каждом запросе к серверу, необходимо указать корневой каталог сервера, например, "path=/".
Если этот атрибут не указан, то значение cookie распространяется только на документы в той же
директории, что и документ, в котором было установлено значение cookie.</li>

<li><b>secure</b> - если стоит этот маркер, то информация cookie пересылается только через
HTTPS (HTTP с использованием SSL - Secure Socket Level), в защищенном режиме. Если этот
маркер не указан, то информация пересылается обычным способом.</li>
</ul>
