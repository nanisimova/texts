# Использование MooseX::Singleton в Catalyst-приложении

*Использование MooseX::Singleton в Catalyst-приложении на конкретном примере, при реализации работы с конфигурационными данными. Использование паттерна "Singleton" в Catalyst-приложении. Примеры кода. Примеры использования Class::Accessor.*

Синтаксис:

```perl
package MyClass;
use MooseX::Singleton;

has config => ( is  => 'rw' );
```

```perl
<pre>package main;

use MyClass;

my $instance = MyClass->instance;

# можно вызвать MyClass->instance еще раз, будет возвращена ссылка 
# на тот же самый объект.
my $same = MyClass->instance;
```

## Пример использования MooseX::Singleton

Задача:

В catalyst-приложении планируется использование внешнего конфигурационного файла. Требуется,
чтобы файл был прочитан только один раз, и в дальнейшем, все модули приложения имели дело с одним
объектом. Для этого:

<ol>
<li>Создаем класс Config, который осуществляет получение конфигурационных данных, их обработку.</li>
<li>Создаем класс SingletonConfig, который реализует идеи паттерна Singleton и позволяет создать объект-одиночку.</li>
<li>Передаем данные конфига Catalyst-приложению.</li>
</ol>

### Config.pm

Config.pm реализует доступ к конфигурационным данным. В приведенном примере, конфиг - это всего лишь хеш. Можно определить содержимое хеша так, как сделано в примере, а можно - взять данные из файла, распарсить и уже потом поместить их в хеш.

```perl
package Config;

use base qw/Class::Accessor/;

__PACKAGE__->mk_accessors(qw/config_data/);

sub new {
    my $class = shift;
    
    my $self = $class->SUPER::new(@_);
    $self->{'config_data'} = { 'dbhost' => 'localhost' };

    return $self;
}
1;
```

#### Что такое Class::Accessor

<font color="#00aa00">Class::Accessor</font> используется для автоматического создания функция-аксессоров. Аксессоры (от англ. access — доступ) - это функции, которые реализуют доступ к внутренним данным объекта.

Т.е. вместо того, чтобы писать для каждого поля объекта код:

```perl
sub name {
    my $self = shift;
    my $value = shift;

    if($value) {
        $self->{name} = $value;
    }

    return $self->{name};
}

sub city {
...
}
```

Можно использовать <font color="#00aa00">Class::Accessor</font> и создать функции-аксессоры одной строкой:
<pre>__PACKAGE__->mk_accessors(qw/name city/);</pre>

<ul>
<li>Метод <font color="#00aa00">mk_accessors()</font> позволяет создать аксессор и для записи данных в поле объекта, и для чтения данных из него.</li>
<li>Чтобы создать аксессор только для чтения данных (getter), можно использовать метод <font color="#00aa00">mk_ro_accessors()</font> .</li>
<li>Для создания аксессоров, которые позволяют только записывать данные, но не получать их (setter), можно использовать метод <font color="#00aa00">mk_wo_accessors()</font> .</li>
</ul>

В дальнейшем, созданный аксессор можно использовать:

```perl
    my $my_obj = MyClass->new({
        'name' => 'value1'
    });

    my $info = $my_obj->name;
    $my_obj->name('value2');
```

Вызов <font color="#00aa00">SUPER::new</font> позволяет использовать унаследованный у <font color="#00aa00">Class::Accessor</font> метод <font color="#00aa00">new()</font>, чтобы создать объект собственного класса.
<pre>my $self = $class->SUPER::new(@_);</pre>

### SingletonConfig.pm

```perl
package SingletonConfig;

use MooseX::Singleton;
use Config;

has config => ( is  => 'rw' );

sub BUILD {
    my $self = shift;

    my $conf_obj = Config->new();
    my $config_data = $conf_obj->config_data;

    $self->config($config_data);
    $self;
}

1;
```

### MyApp.pm

```perl
package MyApp;
use Moose;

use SingletonConfig;

__PACKAGE__->config(
    %{ SingletonConfig->instance->config },
);
```

Метод <font color="#00aa00">instance()</font> - специальная альтернатива методу <font color="#00aa00">new()</font> - либо создает новый объект, либо возвращает ссылку на уже существующий.

<font color="#00aa00">$c-&gt;config</font> (в данном случае <font color="#00aa00">__PACKAGE__-&gt;config</font>) - возвращает или получает ссылку на хеш, который содержит конфигурационные данные для Catalyst-приложения.

### MyApp::Controller::Root

После этого, можно получать доступ к конфигурационным данным из любого модуля.

```perl
package MyApp::Controller::Root;

use uni::perl ':dumper';
use Moose;
use namespace::autoclean;

BEGIN { extends 'Catalyst::Controller' }

sub index :Path :Args(0) {
    my ( $self, $c ) = @_;

    $c->stash->{dbhost} = $c->config->{'dbhost'};
}
1;
```

