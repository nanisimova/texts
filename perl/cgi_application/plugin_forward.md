# CGI::Application. Передача управления от одной подпрограммы к другой в процессе работы приложения. CGI::Application::Plugin::Forward

Иногда, в процессе работы приложения возникает необходимость изменить режим работы приложения, передать управление другой подпрограмме, например, в случае возникновения ошибок.

В <font color="#00aa00">CGI::Application</font> для этого можно использовать одно из двух простых решений.

## Вариант 1

Для передачи управления другой подпрограмме просто вызывается соответствующая подпрограмма. Режим работы приложения при этом не изменяется.

```perl
sub on_start_handler {
	my $self = shift;
	my $tt_params = {};

	my $q = $self->query();
	my $login = $q->param("login");

	unless ($login) {
		return $self->on_login_handler;
	} 

	$tt_params->{block} = 'show_secret_page';
	return $self->tt_process('template2.tt', $tt_params);
}

sub on_login_handler {
	my $self = shift;
	my $tt_params = {};

	# current_runmode будет содержать значение "on_start"
	$tt_params->{current_runmode} = $self->get_current_runmode;

	$tt_params->{block} = 'show_form';
	return $self->tt_process('template2.tt', $tt_params);
}
```

## Вариант 2. CGI::Application::Plugin::Forward

Использовать для передачи управления метод <font color="#00aa00">forward()</font> модуля <font color="#00aa00">CGI::Application::Plugin::Forward</font>.

От предыдущего варианта (<font color="#00aa00">$self-&gt;other_handler</font>) этот отличается тем, что после передачи управления другой подпрограмме, <font color="#00aa00">forward()</font> обновит значение режима работы приложения.

При вызове <font color="#00aa00">$self-&gt;get_current_runmode</font> будет возвращаться значение нового режима работы.

**Синтаксис метода forward()**:

```perl
forward('run_mode_name', @run_mode_params);
```

**Пример использования CGI::Application::Plugin::Forward**:

```perl
package App2;
 
use base 'CGI::Application';
use CGI::Application::Plugin::Forward;
 
sub setup {
	my $self = shift;

	$self->mode_param('step');
	$self->start_mode('on_start');	

	$self->run_modes(
		on_start => \&on_start_handler,
		on_login => \&on_login_handler,
		AUTOLOAD => sub { return 'Запрошенной страницы не существует' }
	);
}

sub on_start_handler {
	my $self = shift;
	my $q = $self->query();
	my $tt_params = {};

	my $login = $q->param("login");
	if ($login) {
		return $self->forward('on_login');
	} 

	$tt_params->{block} = 'show_form';
	return $self->tt_process('template2.tt', $tt_params);
}

sub on_login_handler {
	my $self = shift;
	my $tt_params = {};

	$tt_params->{block} = 'show_secret_page';

	# current_runmode будет содержать значение "on_login"
	$tt_params->{current_runmode} = $self->get_current_runmode;
	return $self->tt_process('template2.tt', $tt_params);
}

```

