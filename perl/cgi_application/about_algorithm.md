# Простое описание алгоритма работы CGI::Application. Методы cgi-скриптов

CGI::Application предоставляет два метода, которые используются в cgi-скриптах: <font color="#00aa00">new()</font> и <font color="#00aa00">run()</font>.

```perl
use App3;
my $app = App3->new;
</pre>
<font color="#00aa00">new()</font> - создает объект CGI::Application.
<pre>$app->run();
```

<font color="#00aa00">run()</font> - выполняет определенный сценарий работы над объектом CGI::Application.


## Алгоритм работы new()

<ol>
<li>создает объект CGI::Application</li>
<li>задает параметры, которые будут использоваться приложением "по-умолчанию".
<pre>
	header_type = 'header'
	mode_param = 'rm'
	start_mode = 'start'
</pre>
</li>
<li>сохраняет переданные методу new параметры: QUERY, TMPL_PATH, PARAMS</li>
<li>если определен метод cgiapp_init() - запускает его на выполнение</li>
<li>запускает на выполнение метод setup()</li>
</ol>


## Алгоритм работы run()

<ol>
<li>если определен метод cgiapp_prerun() - запускает его на выполнение</li>
<li>запускает на выполнение метод, один из заданных через run_modes(), либо "по-умолчанию" - start</li>
<li>если определен метод cgiapp_postrun() - запускает его на выполнение</li>
<li>формирует http-заголовки</li>
<li>отправляет результаты работы браузеру клиента</li>
<li>если определен метод teardown() - запускает его на выполнение</li>
<ol>

