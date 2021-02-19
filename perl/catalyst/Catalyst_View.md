# Catalyst::View

Catalyst::View - базовый класс для создания представлений (view) Catalyst.

Пример использования:

```perl
package Catalyst::View::JSON;
use base qw( Catalyst::View );

sub process {
    my($self, $c) = @_;

    # template processing goes here
    # ...

    $c->response->body($content); # или например, м.б. $c->res->output($output);
}
```

Catalyst::View наследует методы Catalyst::Component.

Существует договоренность, что имена шаблонов, которые используются для формирования итоговой страницы и отправки ее пользователю, должны передаваться с помощью <font color="#00aa00">$c-&gt;stash-&gt;{template}</font>.

Теоретически, если вы реализуете собственное представление для каких-либо целей, вы можете использовать любые переменные для передачи данных. Однако, если предполагается выкладывание модуля в общий доступ, лучше придерживаться общих правил.

## Методы Catalyst::View

**process**

При попытке вызова сгенерирует ошибку. Должен быть переопределен в дочерних классах.

Пример переопределенного метода **process**:

```perl
sub process {
    my $self = shift;
    my ($c) = @_;

    my $template = $c->stash->{template};

    # метод render в данном случае занимается формированием 
    # тела и структуры документа
    my $content = $self->render( $c, $template, $c->stash );

    # если мы хотим вернуть пользователю какой-то документ 
    # в отличном от html формате
    $c->res->headers->header( "Content-Type" => "text/csv" )
      unless ( $c->res->headers->header("Content-Type") );

    # отправляем данные пользователю
    $c->res->body($content);
}

sub render {
    # do_something
}
```
