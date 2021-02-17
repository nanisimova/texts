# CGI::Application. Дополнительные методы приложений CGI::Application

## delete()
<pre>$self-&gt;delete('step');</pre>

Метод <font color="#00aa00">delete()</font> удаляет установленный с помощью <font color="#00aa00">param()</font> параметр. Метод действует аналогично методу <font color="#00aa00">delete()</font> модуля CGI.pm .

## dump()
<pre>print STDERR $self-&gt;dump();</pre>

Результат:
<pre>Current Run mode: 'on_get_result'

Query Parameters:
        day =&gt; '19'
        month =&gt; '02'
        nowyear =&gt; '2010'
        step =&gt; 'on_get_result'

Query Environment:
        CHARSET =&gt; 'windows-1251'
        CHARSET_DETERMINED_BY =&gt; 'AcceptCharset'
        CHARSET_HTTP_METHOD =&gt; 'http://'
        CHARSET_REFERER =&gt; 'http://aninatalie/cgi-bin/app.cgi'
...
</pre>
Метод <font color="#00aa00">dump()</font> полезен при отладке программ. Возвращает информацию об окружении и параметрах запроса. Возвращаемая информация отформатирована для удобства восприятия.

## dump_html()
<pre>print STDERR $self-&gt;dump_html();</pre>

Тоже самое, что и <font color="#00aa00">$self-&gt;dump()</font> , но результаты выполнения возвращаются в формате html. Результат выполнения функции можно отсылать на вывод браузеру.

## error_mode()

<pre>sub setup {
	my $self = shift;

	$self-&gt;error_mode('on_error');
	...
}

sub on_error {
	return '&lt;h1&gt;Ошибочка вышла&lt;/h1&gt;'
}

sub on_start {
	my $self = shift;
	die; # вот тут-то и будет вызываться on_error
}

</pre>
Метод <font color="#00aa00">error_mode()</font> назначает специальный режим для ситуаций, когда неожиданно умирает выполняющийся режим.

Метод позволяет перед завершением работы вывести специальную страницу, список ошибок или сделать записи в логи.

## get_current_runmode()

<pre>my $step = $self-&gt;get_current_runmode(); 
# в данном случае, переменная содержала "on_start".
</pre>
Метод <font color="#00aa00">get_current_runmode()</font> возвращает название режима приложения, который реализуется в текущий момент времени. Если метод вызывается слишком рано, и процесс выполнения приложения еще не дошел до момента определенения режима - функция вернет undef.

## header_add()

<pre>$self-&gt;header_add(-type =&gt; 'image/gif');
# браузер отобразил ошибку: Изображение &lt;app.cgi&gt; не может быть показано, 
# так как содержит ошибки. В реальности же передавался html-документ
</pre>

Метод <font color="#00aa00">header_add()</font> используется для добавления / изменения параметров исходящих заголовков запросов. Все передаваемые параметры в конечном счете будут переданы методу <font color="#00aa00">header()</font> модуля CGI.pm. Данный метод удобно использовать для работы с Cookies.

## header_props()

<pre>sub on_set_cookies {
	my $self = shift;
	my $query = $self-&gt;query();

	my $tt_params = {};

	my $auth_cookie = $query-&gt;cookie(
			-name=&gt;'alisa',
			-value=&gt;'pass',
			-expires=&gt;'-1h',
			-path=&gt;'/cgi',
			-domain=&gt;'.aninatalie.ru',
	);

	$self-&gt;header_props(-cookie=&gt;[$auth_cookie]);

	my %headers = $self-&gt;header_props();
	$tt_params-&gt;{'header_props'} = \%headers;

	return $self-&gt;tt_process('template2.tt', $tt_params);
}
</pre>

Метод <font color="#00aa00">header_props()</font> устанавливает HTTP заголовки . Все установленные параметры будут переданы на выполнение методам <font color="#00aa00">header()</font> и <font color="#00aa00">redirect()</font> модуля CGI.pm.

При добавлении заголовков через метод <font color="#00aa00">header_props()</font> надо учитывать, что он обнулит все заголовки, которые были установлены ранее и установит свои. Если требуется добавить заголовки к уже существующим - следует использовать <font color="#00aa00">header_add()</font> .

Если выполнить <font color="#00aa00">header_props()</font> без параметров, он вернет хэш массивов - список всех заголовков, которые запланировано передать клиенту.

## header_type()

<pre>sub on_start {
	my $self = shift;

	$self-&gt;header_type('redirect');
	$self-&gt;header_props(-url=&gt;  "http://aninatalie.ru" );
}
</pre>

С помощью метода <font color="#00aa00">header_type()</font> можно задать заголовки для перенаправления запроса на другой URL или указать, чтоб фреймворк не возвращал серверу вообще никаких заголовков.
<pre>$self-&gt;header_type('none');
# в ответе фреймворк не вернет вообще никаких заголовков, и 
# сервер назначит для использования тип запроса по-умолчанию - text/plane
</pre>

## mode_param()

Данный метод вызывается из метода <font color="#00aa00">setup()</font> и предназначен для вмешательства в процесс определения режима, в котором будет выполняться приложение.

Есть несколько вариантов использования <font color="#00aa00">mode_param()</font> .

1) Методу передается название параметра, значение которого будет определять
режим приложения.
<pre>$self-&gt;mode_param('step');</pre>

2) Методу <font color="#00aa00">mode_param()</font> передается ссылка на подпрограмму, которая определяет режим
по одной ей известным критериям, и возвращает название запускаемого режима.
<pre>sub setup {
	my $self = shift;

	$self-&gt;mode_param(\&amp;get_mode);
	...
}

sub get_mode {
	my $self = shift;
	my $q = $self-&gt;query();

	my $step = $q-&gt;param('step');
	return 'on_get_result' if $step eq 'on_get_result';
	return 'on_start';

}
</pre>

## prerun_mode()

<pre>sub cgiapp_prerun {
        my $self = shift;
	...
        unless ($confirm_flag) {
                $self-&gt;prerun_mode('on_start');
	}
}
</pre>

<font color="#00aa00">prerun_mode()</font> - метод, предназначенный для использования только в рамках <font color="#00aa00">cgiapp_prerun()</font>. Позволяет в последний момент переназначить обработчик режима, который будет запущен сразу после выполнения <font color="#00aa00">cgiapp_prerun()</font>.

