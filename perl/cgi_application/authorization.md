# Простой пример использования CGI::Application::Plugin::Authorization

Модуль <font color="#00aa00">CGI::Application::Plugin::Authorization</font> помогает разделить пользователей на различные группы и дать этим группам разные права на выполнение тех или иных режимов приложения.

## Скрипт cgi

```perl
#!/usr/bin/perl
 
use strict;
use warnings;
 
use App3;
App3->new->run;
```

## Модуль pm

Есть пользователи <font color="#00aa00">user1</font> и <font color="#00aa00">user2</font>. <font color="#00aa00">User1</font> входит в группу <font color="#00aa00">group1</font>, а <font color="#00aa00">user2</font> включен в группу <font color="#00aa00">group2</font>. Пользователям группы <font color="#00aa00">group1</font> разрешено выполнение режима <font color="#00aa00">on_private1</font>, всем остальным - запрещено.

Все это реализовано поверх систем аутентификации и сессий. Пример простой, без использования баз данных и пр., для максимальной наглядности.

```perl
package App3;
use strict;
use warnings;
 
use base 'CGI::Application';
use CGI::Application::Plugin::TT;

use CGI::Application::Plugin::Session;
use CGI::Application::Plugin::Authentication;
use CGI::Application::Plugin::Forward;
use CGI::Application::Plugin::Authorization;
 
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
    on_forbidden => \&forbidden,

    AUTOLOAD => sub { return 'Запрошенной страницы не существует' }
  );
}

sub cgiapp_init {
  my $self = shift;
 
  $self->session_config(
    CGI_SESSION_OPTIONS => [ "driver:File", $self->query, {Directory=>'/dev-lab/tmp'} ],
    DEFAULT_EXPIRY      => '+1w',
    COOKIE_PARAMS       => {
      -expires => '+24h',
      -path    => '/',
    },
    SEND_COOKIE         => 1,
  );

  $self->authen->config(
    DRIVER  => [ 'Generic', { user1 => 'password123', user2 => 'password12345' } ],
    STORE	=> 'Session',
    LOGOUT_RUNMODE		=> 'on_logout',
    LOGIN_RUNMODE		=> 'on_login',
    POST_LOGIN_RUNMODE	=> 'on_ok',
  );

  $self->authen->protected_runmodes(
    'on_private1',
  );

  # список пользователей и определение их принадлежности к группам
  my %user_group = (
    user1 => 'group1',
    user2 => 'group2',
  );

  # конфигурирование системы авторизации
  $self->authz->config(
    DRIVER => [ 'Generic', sub {
      my ($username,$group) = @_;
      return ($user_group{$username} eq $group);
    } ],
    # обработчик, который выполнится при попытке 
    # нелегального доступа к защищенным режимам
    FORBIDDEN_RUNMODE => 'on_forbidden',
  );

  # к режиму on_private1 разрешен доступ только представителям group1
  $self->authz->authz_runmodes(
    on_private1 => 'group1',
  );
}

# подпрограмма будет выполнена при попытке нелегального доступа
# к защищенным режимам
sub forbidden {
  my $self = shift;

  my $tt_params = {};
  $tt_params->{block} = 'show_forbidden';

  return $self->tt_process('template3.tt', $tt_params);
}

# выполняется при успешной аутентификации пользователя
sub ok {
  my $self = shift;

  my $tt_params = {};
  $tt_params->{block} = 'show_ok_page';

  return $self->tt_process('template3.tt', $tt_params);
}

# предлагает форму аутентификации. Если пользователь уже опознан,
# перенаправит на секретную страницу.
sub start {
  my $self = shift;
  my $q = $self->query();
  my $tt_params = {};
	
  my $user = $self->authen->username;

  if ($user) {
    return $self->forward('on_private1');
  }

  $tt_params->{block} = 'show_index_page';
  return $self->tt_process('template3.tt', $tt_params);
}

# выводит секретную страницу
sub private1 {
  my $self = shift;
  my $tt_params = {};

  $tt_params->{block} = 'show_private_page';

  return $self->tt_process('template3.tt', $tt_params);
}

# завершает сеанс работы, удаляет сессию
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

# процесс аутентификации
sub login {
  my $self = shift;

  if ($self->authen->username) {
    # если аутентификация успешна - перенаправляем на режим on_private1
    return $self->forward('on_private1');
  } else {
    # если аутентификация не успешна - предлагаем повторить попытку
    return $self->forward('on_start');
  }
}

```

## Шаблон tt

```html
<meta http-equiv="Content-Type" content="text/html; charset=windows-1251">
<title>session</title>
...

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

[%-#-----------------------------------------%]
[%-BLOCK show_forbidden-%]
[%-#-----------------------------------------%]

Forbidden: Вам туды нельзя

<form action="app3.cgi" method="POST">
<input type="hidden" name="step" value="on_logout">
<input value="Выйти" type="submit">
</form>

[%-#-----------------------------------------%]
[%-END-%]
```


