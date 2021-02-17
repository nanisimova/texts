# Простой пример использования CGI::Application::Plugin::Authentication

Это очень простой пример, с минимально возможным использованием различных дополнительных модулей.

Хорошо подходит для демонстрации работы <font color="#00aa00">CGI::Application::Plugin::Authentication</font>, алгоритм которого, кстати, весьма не очевиден. Впрочем, возможно, требуется только привычка.

## Скрипт cgi

```perl
#!/usr/bin/perl
 
use strict;
use warnings;
use App3;

App3->new->run;
```

## Модуль pm

```perl
package App3;
use strict;
use warnings;
 
use base 'CGI::Application';
use CGI::Application::Plugin::TT;

use CGI::Session::Driver::file;
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
 
  # после аутентификации в сессию будет добавлена информация о клиенте
  $self->session_config(
    CGI_SESSION_OPTIONS => [ 
      "driver:File", 
      $self->query, 
      {Directory=>'/dev-lab/tmp'} 
    ],
    DEFAULT_EXPIRY      => '+1w',
    COOKIE_PARAMS       => {
      -expires => '+24h',
      -path    => '/',
    },
    SEND_COOKIE         => 1,
  );

  # задаем настройки аутентификации
  $self->authen->config(
    DRIVER  => [ 'Generic', { user1 => '123' } ],
    STORE	=> 'Session',
    LOGOUT_RUNMODE		=> 'on_logout',
    LOGIN_RUNMODE		=> 'on_login',
    POST_LOGIN_RUNMODE	=> 'on_ok',
  );

  # эти режимы будут доступны только прошедшим аутентификацию клиентам
  $self->authen->protected_runmodes(
    'on_private1',
  );
}

# подпрограмма выполнится, если клиент указал корректные логин и пароль
sub ok {
  my $self = shift;

  my $tt_params = {};
  $tt_params->{block} = 'show_ok_page';

  return $self->tt_process('template3.tt', $tt_params);
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
  return $self->tt_process('template3.tt', $tt_params);
}

# если клиент не опознан, вместо этой подпрограммы сработает login()
sub private1 {
  my $self = shift;
  my $tt_params = {};

  $tt_params->{block} = 'show_private_page';

  return $self->tt_process('template3.tt', $tt_params);
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

  return $self->tt_process('template3.tt', $tt_params);
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
## Шаблон tt

```html
<meta http-equiv="Content-Type" content="text/html; charset=windows-1251">
<title>session</title>
<h1>Session</h1>

[%- TRY -%]
	[%-INCLUDE $block-%]
[%- CATCH -%]
	Ошибочка вышла
[%- END -%]

[%-#-----------------------------------------%]
[%-BLOCK show_index_page-%]
[%-#-----------------------------------------%]

<form action="app3.cgi" method="POST">
<input type="hidden" name="step" value="on_login">
	
Логин
<input type="text" size="10" name="authen_username" maxlength="40">

Пароль
<input type="text" size="10" name="authen_password" maxlength="20">

<input value="Войти" type="submit">
</form>

[%-#-----------------------------------------%]
[%-END-%]

[%-#-----------------------------------------%]
[%-BLOCK show_private_page-%]
[%-#-----------------------------------------%]

Эта страница только для авторизованных пользователей

<form action="app3.cgi" method="POST">
<input type="hidden" name="step" value="on_logout">
<input value="Выйти" type="submit">
</form>

[%-#-----------------------------------------%]
[%-END-%]


[%-#-----------------------------------------%]
[%-BLOCK show_close_session_page-%]
[%-#-----------------------------------------%]

Ваша сессия завершена. До свидания!
[%-#-----------------------------------------%]
[%-END-%]

[%-#-----------------------------------------%]
[%-BLOCK show_ok_page-%]
[%-#-----------------------------------------%]

Вы успешно авторизованы! Можете посмотреть специальные страницы.
<a href="app3.cgi?step=on_private1">Private page</a>

[%-#-----------------------------------------%]
[%-END-%]

```

## А тем временем... Содержимое сессионного файла

В приведенном примере, сессионные данные хранятся в файле, который создается по адресу /dev-lab/tmp. После аутентификации, в этот же самый файл помещается информация о ней.

<font color="#00aa00">cgisess_855e6124edd2c938d06bb9b837e1075b</font>:

```html
$D = {
  '_SESSION_ID' => '855e6124edd2c938d06bb9b837e1075b',
  '_SESSION_ETIME' => 604800,
  '_SESSION_REMOTE_ADDR' => '127.0.0.1',
  '_SESSION_CTIME' => 1290694127,
  '_SESSION_ATIME' => 1290694135,
  '_SESSION_EXPIRE_LIST' => {},
  'AUTH_LAST_LOGIN' => 1290694135,
  'AUTH_LAST_ACCESS' => 1290694135,
  'AUTH_USERNAME' => 'user1',
  'AUTH_LOGIN_ATTEMPTS' => 0
}
;;$D
```



