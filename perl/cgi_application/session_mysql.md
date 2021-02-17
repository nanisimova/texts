# Пример использования CGI::Application::Plugin::Session. Хранение сессии в БД mysql

Данный пример демонстрирует создание сессии и ее хранение в базе данных <font color="#00aa00">mysql</font>.

Пример основан на <a href="session.md">предыдущем варианте кода</a>, поэтому я привожу тут только тот код, который был изменен для взаимодействия с БД.

Для хранения данных сессии в базе данных, прежде всего требуется создать базу данных и специальную таблицу:
<pre>CREATE TABLE `sessions` (
  `id` char(32) NOT NULL,
  `a_session` text NOT NULL,
  PRIMARY KEY (`id`)
);
</pre>
При создании новой сессии, в БД будет добавлена новая запись. Поле <font color="#00aa00">id</font> будет содержать сессионный ключ, <font color="#00aa00">a_session</font> - сессионные данные.

Пример:
<pre>
INSERT INTO `sessions` (`id`, `a_session`) VALUES
('12a2ebe8df57f3c89869ae91fe24b86a',
'$D = {''_SESSION_ID'' =&gt; ''12a2ebe8df57f3c89869ae91fe24b86a'',
''_SESSION_ETIME'' =&gt; 604800,
''_SESSION_ATIME'' =&gt; 1295775623,
''_SESSION_EXPIRE_LIST'' =&gt; {},
''_SESSION_REMOTE_ADDR'' =&gt; ''191.96.20.01'',
''_SESSION_CTIME'' =&gt; 1295768464};;$D');
</pre>

Пример кода:

```perl
package App3;
use strict;
use warnings;

use base 'CGI::Application';
use CGI::Application::Plugin::TT;

use CGI::Application::Plugin::DBH (qw/dbh_config dbh/);

use CGI::Session::Driver::mysql;
use CGI::Application::Plugin::Session;

sub setup {
  my $self = shift;

  $self->mode_param('step');
  $self->start_mode('on_start');	

  $self->run_modes(
    on_start => \&on_start,
    on_save_addr => \&on_save_addr,
    on_close_session => \&on_close_session,
    AUTOLOAD => sub { return 'Запрошенной страницы не существует' }
  );
}

sub cgiapp_init {
  my $self = shift;

  $self->dbh_config("DBI:mysql:database=db1;host=localhost",
    'user', 'password');

  # Настройки работы с сессиями
  $self->session_config(
    CGI_SESSION_OPTIONS => [ "driver:mysql", $self->query, {Handle=>$self->dbh} ],
    DEFAULT_EXPIRY      => '+1w',
    COOKIE_PARAMS       => {
      -expires => '+24h',
      -path    => '/',
    },
    SEND_COOKIE         => 1,
  );
}

sub on_start {
  my $self = shift;
  my $q = $self->query();
  my $tt_params = {};

  $tt_params->{addr} = $self->session->param('addr');
  $tt_params->{block} = 'show_index_page';

  return $self->tt_process('template3.tt', $tt_params);
}

```

