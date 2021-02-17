# CGI::Application. Замещаемые методы CGI::Application

*Частичный перевод CGI::Application  / Methods to possibly override . Третья из четырех частей перевода. Остальные будут опубликованы позднее. Примеры кода - авторские.*

При желании, Вы можете произвести замещение перечисленных методов родительского класса своими собственными. Это расширит Ваши возможности в использовании и конфигурировании <font color="#00aa00">CGI::Application</font>.

Почти всегда перезаписывается метод <font color="#00aa00">setup()</font>. Все остальные методы - замещаются достаточно редко.

Если заглянуть в исходный код модуля <font color="#00aa00">Application.pm</font>, можно заметить, что родительские методы <font color="#00aa00">teardown(), cgiapp_init(), cgiapp_prerun() и cgiapp_postrun()</font> определены, вызываются в нужный момент, но никакой работы не выполняют:
<pre>Application.pm:
...
sub cgiapp_prerun {
  my $c = shift;
  my $rm = shift;

  # Nothing to prerun, yet!
}
...
</pre>
Эти методы предназначены именно для удобства разработчиков.

## setup()

Этот метод вызывается методом <font color="#00aa00">new()</font> автоматически. Метод <font color="#00aa00">setup()</font> используется для задания конфигурации приложения.

В рамках <font color="#00aa00">setup()</font> возможен вызов специальных методов:
<ul>
 	<li>mode_param();</li>
 	<li>start_mode();</li>
 	<li>error_mode();</li>
 	<li>run_modes();</li>
 	<li>tmpl_path().</li>
</ul>
Чаще всего, <font color="#00aa00">setup()</font> используется только для того, чтобы задать список всех возможных режимов работы приложения и определить режим, который будет запускаться по-умолчанию.

Пример:
<pre>sub setup {
  my $self = shift;

  $self-&gt;mode_param('step');
  $self-&gt;start_mode('on_start');	

  $self-&gt;run_modes(
    on_get_result	=&gt; 'on_get_result',
    on_start	=&gt; 'start',
      AUTOLOAD	=&gt; sub { return 'Запрошенной страницы не существует' }
  );
}
</pre>

## teardown()

Метод вызывается после выполнения обработчиков режимов приложения.
<pre>sub teardown {
  my $self = shift;

  print STDERR Dumper($self)."\n\n";
}
</pre>

## cgiapp_init()

<pre>sub cgiapp_init  {
  my $self = shift;

  $self-&gt;dbh_config('DBI:mysql:database=db2;host=localhost', 'user', 'pass');
}
</pre>
Метод <font color="#00aa00">cgiapp_init()</font> вызывается на выполнение перед вызовом метода <font color="#00aa00">setup()</font>. Метод получает в свое распоряжение все параметры, которые были переданы методу <font color="#00aa00">new()</font>.

## cgiapp_prerun()

Метод <font color="#00aa00">cgiapp_prerun()</font> вызывается перед запуском одного из режимов работы (run mode) приложения.
<pre>sub cgiapp_prerun {
  my $self = shift;
  $self-&gt;tt_params(block =&gt; 'show_form');
}

</pre>
Метод <font color="#00aa00">cgiapp_prerun()</font> получает на вход те же данные, которые передаются затем обработчикам режимов. <font color="#00aa00">Cgiapp_prerun()</font> вызывается только, если определен в коде.

## cgiapp_postrun()

Метод <font color="#00aa00">cgiapp_postrun()</font> вызывается на выполнение после
того, как будут выполнены обработчики режимов, но до того, как начнут формироваться HTTP-заголовки.

<font color="#00aa00">Cgiapp_postrun()</font> вызывается только, если определен в коде приложения.

Метод удобен, если требуется дополнительно обработать возвращаемые данные (например, при работе с шаблонизаторами) или внести изменения в HTTP-заголовки (например, организовать переадресацию).

На вход <font color="#00aa00">cgiapp_postrun()</font> получает ссылку на объект CGI::Application и ссылку на возвращенные обработчиком режима данные.
<pre>use Data::Dumper;
...
sub cgiapp_postrun {
  my $self = shift;
  my $output_ref = shift;

  print STDERR Dumper($output_ref);
}
...
</pre>

## cgiapp_get_query()

Стандартный способ получения переданных клиентом параметров - это использование метода <font color="#00aa00">query()</font> модуля <font color="#00aa00">CGI.pm</font>.

Замещение метода <font color="#00aa00">cgiapp_get_query()</font> позволяет использовать для получения параметров иные способы, например, с помощью модуля <font color="#00aa00">CGI::Simple</font> или, если у Вы работаете с mod_perl, <font color="#00aa00">CGI::Application::Plugin::Apache</font>.

Главное, чтобы соблюдалась некоторая совместимость с API <font color="#00aa00">CGI.pm</font>, в частности, с методом <font color="#00aa00">param() CGI.pm</font>.
<pre>use CGI::Simple;

sub cgiapp_get_query {
  my $self = shift;

  return CGI::Simple-&gt;new();
}

sub on_start {
  my $self = shift;
  my $q = $self-&gt;query();

  my $login = $q-&gt;param("login");
...
}
</pre>
