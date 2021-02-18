# Краткое описание Apache::Template

## Apache::Template

<font color="#00aa00">Template Toolkit</font> - мощная perl-библиотека для работы с шаблонами, позволяющая разделять код, данные и представление. (Система обработки шаблонов). Фактически, Template Toolkit - это набор модулей.

<font color="#00aa00">Template.pm</font> - фронтенд к Template Toolkit, предоставляющий доступ к возможностям библиотеки через один модуль с простым интерфейсом. <font color="#00aa00">Модуль Template</font> создает и использует объект Template::Service для передачи и вывода данных по заданному пути (STDOUT, файл, и т.д.). <font color="#00aa00">Template.pm</font> работает с STDOUT, ссылками на переменные и т.д.

Модуль <font color="#00aa00">Apache::Template</font> - это фронтенд к Template Toolkit, который создает объект <font color="#00aa00">Template::Service::Apache</font>, вызывает его при необходимости и отправляет вывод обратно в подходящий объект Apache::Request. <font color="#00aa00">Apache::Template</font> работает с mod_perl,  Apache::Request и конфигурацией, переданной через httpd.conf.

Всю основную работу для обоих фронтендов выполняет <font color="#00aa00">Template::Service</font> и его подклассы.

В зависимости от ситуации и специфики окружения следует использовать <font color="#00aa00">Template.pm</font> или <font color="#00aa00">Apache::Template</font>.


## Синтаксис Apache::Template

<pre>
    PerlModule          Apache::Template

    TT2Trim             On
    TT2PostChomp        On
    TT2EvalPerl         On
    TT2IncludePath      /usr/local/tt2/templates
    TT2IncludePath      /home/abw/tt2/lib
    TT2PreProcess       config header
    TT2PostProcess      footer
    TT2Error            error

    &lt; Files *.tt2 &gt;
        SetHandler      perl-script
        PerlHandler     Apache::Template
    &lt; / Files &gt;

    &lt; Location /tt2 &gt;
        SetHandler      perl-script
        PerlHandler     Apache::Template
    &lt; / Location &gt;

</pre>

## Использование Template Toolkit в коде статичных страниц (html, etc.)

В httpd.conf задается конфигурация:
<pre>
PerlModule          Apache::Template

TT2IncludePath	/www.aninatalie.ru/templates
TT2PluginBase	Template_Plugin

&lt; Files *.shtml &gt;
     SetHandler      perl-script
     PerlHandler     Apache::Template
&lt; / Files &gt;
</pre>

Благодаря приведенным инструкциям, теперь, в html-коде файлов с расширением .shtml  можно использовать директивы Template Toolkit.

Например, catalogue_janssen.shtml:

<pre>[% WRAPPER 'page_design.wr' 
	title = 'Каталог товаров Spa Cosmetics - JANSSEN'
%]

&lt; h1 &gt;JANSSEN&lt; /h1 &gt;
&lt; table class="catalogue" &gt;

&lt; tr &gt;
&lt; td &gt;
...

</pre>
При обработке шаблонов будут использоваться параметры, заданные директивами с префиксом TT2, в конфигурационном файле Apache.

