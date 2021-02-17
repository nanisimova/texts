# Использование CGI::Application::Plugin::DBH. Краткий обзор, примеры кода

Терминология

<font color="#00aa00">Дескриптор - handle</font> - это уникальный номер, который используется для идентификации объектов. Обычно дескриптор используется при работе с API, при этом смысл значения дескриптора скрыт за этим API.

Смысл существования дескрипторов в том, чтобы как-то различать единообразные объекты между собой. Например, при открытии нескольких файлов для чтения, или при установлении нескольких соединений с базами данных.

<font color="#00aa00">CGI::Application::Plugin::DBH</font> - используется для простого доступа к дескриптору <font color="#00aa00">DBI - dbh</font>. Особенность работы с <font color="#00aa00">CGI::Application::Plugin::DBH</font> состоит в том, что соединение с заранее определенной БД не устанавливается до тех пор, пока к этой БД не будет сделан хотя бы один запрос. Если приложение отработает без обращений к БД с запросами (например, на ранней стадии будет обнаружена ошибка и выполнение приложения будет прервано) - соединение не будет установлено. Экономия времени выполнения и ресурсов сервера очевидна.

## Методы

### dbh

<pre>my @row_ary = $self-&gt;dbh-&gt;selectrow_array("SELECT * FROM table_name LIMIT 1");</pre>

Метод возвращает дескриптор базы данных DBI. Дескриптор создается при первом вызове данного метода, и просто возвращается при последующих обращениях к методу.

Если требуется использовать сразу несколько разных баз данных и, соответственно, дескрипторов, можно назначать для дескрипторов разные имена, и в дальнейшем обращаться к дескриптору по имени.
<pre>use CGI::Application::Plugin::DBH (qw/dbh_config dbh/);
...

sub cgiapp_init  {
  my $self = shift;

  $self-&gt;dbh_config('first_db_handle', 
    ['DBI:mysql:database=bd1;host=localhost', 'user1', 'super_parol']);
  $self-&gt;dbh_config('second_db_handle',
    ["DBI:mysql:database=bd2;host=localhost", 'user2', 'prosto_parol']);
}
...

sub on_start {
  my $self = shift;

  my $tt_params = {
    block =&gt; 'show_form',
  };

  my @row_ary = $self-&gt;dbh('first_db_handle')-&gt;selectrow_array("SELECT * FROM 
  any_table LIMIT 1");
  $tt_params-&gt;{'info'} = \@row_ary;


  my $dbh2 = $self-&gt;dbh('second_db_handle');
  my @row_ary2 = $dbh2-&gt;selectrow_array("SELECT * FROM some_table LIMIT 1");
  $tt_params-&gt;{'info2'} = \@row_ary2;

  return $self-&gt;tt_process('template2.tt', $tt_params);
}
</pre>

+++

<pre>my ($login) = $self-&gt;dbh-&gt;selectrow_array(
  "SELECT login FROM auth WHERE contract_id=$contract_id LIMIT 1");
</pre>

+++

<pre>my $rows = $self-&gt;dbh-&gt;do("INSERT INTO auth (login, pass) VALUES (?, ?)", 
  undef, ($login, $pass));
</pre>

+++

<pre>my $sql = "UPDATE auth SET contract_id=$contract_id WHERE login=$login";
my $sth = $self-&gt;dbh-&gt;prepare($sql);
$sth-&gt;execute;
</pre>

### dbh_config

<pre>sub cgiapp_init  {
  my $self = shift;

  $self-&gt;dbh_config("DBI:mysql:database=main_bd;host=localhost", 
    'main_user', 'main_user_password');
}
</pre>

Метод dbh_config используется для установки параметров подключения к БД. Рекомендуется использовать метод внутри cgiapp_init. Главное, чтобы этот метод был выполнен задолго до того, как произойдет первый вызов dbh() , иначе все закончится очень печально - "500. Ошибочка вышла".

### dbh_default_name

Метод dbh_default_name() указывает название дескриптора, который будет возвращаться "по-умолчанию", если имя дескриптора при вызове метода dbh() не указано. Используется при работе с двумя и более дескрипторами в одном приложении - очень удобно.

Если задать метод без параметров - вернет имя дескриптора, который назначен для вызова "по-умолчанию".
<pre>use CGI::Application::Plugin::DBH (qw/dbh_config dbh dbh_default_name/);

sub setup {
  my $self = shift;

  $self-&gt;dbh_default_name('first_db_handle');
  $self-&gt;dbh_config('first_db_handle', 
    ['DBI:mysql:database=bd1;host=localhost', 'user1', 'super_parol']);
  $self-&gt;dbh_config('second_db_handle',
    ["DBI:mysql:database=bd2;host=localhost", 'user2', 'prosto_parol']);

  $self-&gt;mode_param('step');
...
}
...

sub on_start {
  my $self = shift;

  my @row_ary = $self-&gt;dbh-&gt;selectrow_array("SELECT * FROM any_table LIMIT 1");

  my $dbh2 = $self-&gt;dbh('second_db_handle');
  my @row_ary2 = $dbh2-&gt;selectrow_array("SELECT * FROM some_table LIMIT 1");
...
}
</pre>

