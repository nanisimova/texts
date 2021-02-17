# Обзор Apache::PerlRun

<font style="color: #2f6fab;">Apache::PerlRun</font> - это модуль Apache, который эмулирует CGI-окружение для выполнения Perl-скриптов. Основные принципы работы аналогичны <font style="color: #2f6fab;">Apache::Registry</font>.

Скомпилированные скрипты не кешируются, и при каждом запросе считываются и компилируются заново.

Для работы с <font style="color: #2f6fab;">Apache::Registry</font> скрипты должны соответствовать некоторым требованиям, чтоб избежать ошибок при выполнении и утечек памяти. Для работы под <font style="color: #2f6fab;">Apache::PerlRun</font> изменять скрипты не требуется, т.к. память очищается после выполнения каждого запроса.

Подключение <font style="color: #2f6fab;">Apache::PerlRun</font>, httpd.conf:

<pre>
Alias /cgi/ /perl/apache/scripts/ 
PerlModule Apache::PerlRun

...
&lt; Location /cgi/&gt;
    SetHandler perl-script
    PerlHandler Apache::PerlRun
    Options +ExecCGI
&lt; /Location&gt;
</pre>