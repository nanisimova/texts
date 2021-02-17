# CGI::Application. Использование CGI::Application::Plugin::Redirect

## Краткое описание CGI::Application::Plugin::Redirect

<font color="#00aa00">CGI::Application::Plugin::Redirect</font> - это простой способ сделать перенаправление.

Чтобы выполнить redirect используется метод <font color="#00aa00">redirect($url, $status)</font>.

<font color="#00aa00">$url</font> - это адрес, куда должно произойти перенаправление.

<font color="#00aa00">$status</font> - необязательный параметр, указание которого приведет к выводу страницы с сообщением для пользователя и ссылкой, при переходе по которой и будет осуществлен redirect.

## Пример использования CGI::Application::Plugin::Redirect

```perl
package App2;
use strict;
 
use base 'CGI::Application';
use CGI::Application::Plugin::Redirect;
...

sub on_start {
	my $self = shift;
	...

	unless ($login) {
		$self->header_add(-type => 'text/html');
		return $self->redirect('http://dev-lab.info/auth/', '403 Forbidden');
	} elsif ($login eq "Nadya") {
		return $self->redirect('http://dev-lab.info/secret_block/');
	} else {
		...
	}
...
}
```


