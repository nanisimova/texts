# CGI::Application::Plugin::Authentication. Получение данных о пользователе из БД mysql

После того, как хранение сессионных данных было перенесено в базу данных <font color="#00aa00">mysql</font>, следующим логичным шагом стала реализация получения аутентификационных данных из БД <font color="#00aa00">mysql</font>.

Пример основан на <a href="authorization.md">предыдущем варианте кода</a>. Скрипт <font color="#00aa00">cgi</font> и содержимое шаблона <font color="#00aa00">tt</font> остались неизменными. Изменения вносились только в модуль <font color="#00aa00">App2.pm</font> (в предыдущем примере это был App3.pm), в блок <font color="#00aa00">cgiapp_init</font>.

Итак, что нужно сделать, чтобы заставить работать <font color="#00aa00">CGI::Application::Plugin::Authentication</font> с <font color="#00aa00">mysql</font>?


## 1. Создать в базе данных новую таблицу

Название таблицы и ее полей может быть произвольным.
<pre>CREATE TABLE `db`.`user` (
`name` VARCHAR( 255 ) NOT NULL ,
`password` VARCHAR( 255 ) NOT NULL ,
PRIMARY KEY ( `name` )
) ENGINE = MYISAM ;
</pre>


## 2. Добавить в таблицу список пользователей</h3>

Пароль <a href="authentication_2/">рекомендуется хранить в зашифрованном виде</a>, но пока, для простоты тестирования, пусть хранится "как есть".
<pre>INSERT INTO `db`.`user` (`name`, `password`)
VALUES ('User1', '123');
</pre>

## 3. Настроить подключение к таблице БД в модуле программы

```perl
package App2;
use strict;
use warnings;

use base 'CGI::Application';
use CGI::Application::Plugin::TT;

use CGI::Application::Plugin::DBH (qw/dbh_config dbh dbh_default_name/);

use CGI::Session::Driver::mysql;

use CGI::Application::Plugin::Session;
use CGI::Application::Plugin::Authentication;
use CGI::Application::Plugin::Forward;

sub setup {
  my $self = shift;

  $self->mode_param('step');
  $self->start_mode('on_start');
  $self->run_modes(
    on_start => \&start,
    on_logout => \&logout,
    on_login => \&login,
    on_ok => \&ok,
    on_private1 => \&private1,
    AUTOLOAD => sub { return 'Запрошенной страницы не существует' }
  );
}

sub cgiapp_init {
  my $self = shift;

  $self->dbh_config("DBI:mysql:database=db;host=localhost",
    'user_db', 'password_db');

  # после аутентификации в сессию будет добавлена информация о клиенте
  $self->session_config(
    CGI_SESSION_OPTIONS => [ "driver:mysql", $self->query, {Handle=>$self->dbh} ],
    DEFAULT_EXPIRY      => '+1w',
    COOKIE_PARAMS       => {
      -expires => '+24h',
      -path    => '/',
    },
    SEND_COOKIE         => 1,
  );

  # задаем настройки аутентификации
  $self->authen->config(
    DRIVER => [ 'DBI',
      TABLE       => 'user',
      # CONSTRAINTS - список условий для поиска аутентифицируемого пользователя 
      # в таблице БД
      CONSTRAINTS => {
        'user.name'     => '__CREDENTIAL_1__',
        'user.password' => '__CREDENTIAL_2__',
      },
    ],
    STORE	=> 'Session',
    LOGOUT_RUNMODE		=> 'on_logout',
    LOGIN_RUNMODE		=> 'on_login',
    POST_LOGIN_RUNMODE	=> 'on_ok',
    # ниже, соответственно, CREDENTIAL_1 и CREDENTIAL_2
    CREDENTIALS => [ 'authen_username', 'authen_password' ],
  );

  # эти режимы будут доступны только прошедшим аутентификацию клиентам
  $self->authen->protected_runmodes(
    'on_private1',
  );
}

# далее в код никаких изменений не вносилось. Он оставлен для целостности примера

# подпрограмма выполнится, если клиент указал корректные логин и пароль
sub ok {
  my $self = shift;

  my $tt_params = {};
  $tt_params->{block} = 'show_ok_page';

  return $self->tt_process('template2.tt', $tt_params);
}

sub start {
  my $self = shift;
  my $q = $self->query();
  my $tt_params = {};

  my $user = $self->authen->username;

  if ($user) {
    # клиент опознан. Можно показать что-то особенное
    return $self->forward('on_private1');
  }

  # клиент не опознан, выводим форму для аутентификации
  $tt_params->{block} = 'show_index_page';
  return $self->tt_process('template2.tt', $tt_params);
}

# если клиент не опознан, вместо этой подпрограммы сработает login()
sub private1 {
  my $self = shift;
  my $tt_params = {};

  $tt_params->{block} = 'show_private_page';

  return $self->tt_process('template2.tt', $tt_params);
}

# завершение сеанса работы
sub logout {
  my $self = shift;
  my $tt_params = {};

  if ($self->authen->username) {
    $self->authen->logout;
    $self->session_delete;
  }
  $tt_params->{block} = 'show_close_session_page';

  return $self->tt_process('template2.tt', $tt_params);
}

# аутентификация
sub login {
  my $self = shift;

  if ($self->authen->username) {
    # клиент опознан. Можно показать что-то особенное
    return $self->forward('on_private1');
  } else {
    # клиент не опознан. Оn_start выведет ему
    # форму для аутентификации
    return $self->forward('on_start');
  }
}
1;
```

