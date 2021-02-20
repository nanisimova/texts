# Шаблон проектирования Facade

Шаблон проектирования <font color="#00aa00">Facade</font> используется для предоставления простого в использовании интерфейса к сложному набору интерфейсов и подсистем. Шаблон позволяет скрыть детали реализации систем, упрощая работу с конечным продуктом.

## Краткое описание

Допустим, у нас есть некоторое программное обеспечение, которое предоставляет доступ к системе компиляции программного кода. Каждый компилятор имеет множество подклассов: лексический анализатор, синтаксический анализатор, и т.п. Возможно, некоторым специализированным программам может понадобиться прямой доступ к этим подклассам, но в большинстве случаев, клиентам компилятора знание таких подробностей не требуется - необходимо просто скомпилировать заданный программный код.

Поэтому, имеет смысл предоставить клиентам удобный интерфейс, который будет изолировать классы компилятора от клиента. В систему просто включается дополнительный класс <font color="#00aa00">ProgramRunner</font>. Он определяет унифицированный интерфейс ко всем возможностям компилятора. В этом случае, класс <font color="#00aa00">ProgramRunner</font> будет выступать в роли фасада - предлагать более простой интерфейс к достаточно сложной подсистеме.

Клиенты общаются с подсистемой, посылая запросы фасаду. Он переадресует их подходящим объектам внутри подсистемы.

До применения паттерна:

```perl
my $compiler = Compiler->new();
my $bytecode = $compiler->compile('file');

my $interpeter = Interpreter->new();
$interpreter->execute($bytecode)
```

После применения паттерна:

```perl
my $runner = ProgramRunner->new();
$runner->run('file')
```

## Пример "из жизни"

Паттерн <font color="#00aa00">Facade</font> используется в таких популярных системах, как <font color="#00aa00">LWP</font> и <font color="#00aa00">DBI</font>.

### LWP

Для того, чтобы получить простой файл с сайта, надо установить соединение с этим ресурсом, создать корректный http-запрос, получить http-ответ, разобрать ответ и извлечь из него нужные данные. Если требуется обработать cookies, отправить данные из html-формы, количество работы становится еще больше. Для выполнения всей этой работы требуется подключить множество классов и оперировать множеством объектов. Если в рамках программы требуется реализовать всего лишь разовое подключение к сайту, это как минимум не удобно и не эффективно.

Модуль <font color="#00aa00">LWP</font> представляет собой фасад ко множеству объектов, которые реализуют всю работу по подключению к нужному ресурсу, отправке запросов и получению ответов.

Можно упростить работу еще больше - <font color="#00aa00">LWP::Simple</font> представляет собой фасад для <font color="#00aa00">LWP::UserAgent</font> и др.

Пример:

```perl
use LWP::Simple qw(get);

my $url = "http://dev-lab.info";
my $data = get($url);
```

Кроме того, <font color="#00aa00">LWP</font> является фасадом для использования различных протоколов <font color="#00aa00">HTTP</font>, <font color="#00aa00">HTTPS</font>, <font color="#00aa00">FTP</font>, <font color="#00aa00">NNTP</font>, и разработчику не приходится тратить время на подключение различных библиотек и изучение особенностей работы с ними.

### DBI

Архитектура <font color="#00aa00">DBI</font>:
<pre>
            |&lt;- Scope of DBI -&gt;|
                  .-.   .--------------.   .-------------.
  .-------.       | |---| XYZ Driver   |---| XYZ Engine  |
  | Perl  |       | |   `--------------'   `-------------'
  | script|  |A|  |D|   .--------------.   .-------------.
  | using |--|P|--|B|---|Oracle Driver |---|Oracle Engine|
  | DBI   |  |I|  |I|   `--------------'   `-------------'
  | API   |       | |...
  |methods|       | |... Other drivers
  `-------'       | |...
                  `-'
</pre>

<font color="#00aa00">DBI</font> обеспечивает простой интерфейс доступа к различным серверам баз данных. При этом не нужно беспокоиться о реализации подключения к базе данных, реализации протоколов взаимодействия и формате данных.

Можно даже изменить тип базы данных и ограничиться внесением минимальных изменений в свой код, или даже вообще обойтись без них.

### Class::Facade

Этот модуль реализует простой класс - <font color="#00aa00">Facade</font>, который позволяет создавать объекты, делегирующие выполнение работы другим классам и объектам.

Синтаксис <font color="#00aa00">Class::Facade</font>:

```perl
use Class::Facade;

my $facade = Class::Facade->new({
    method1 => sub { ... },
    method2 => [ $class, $method, $arg1, $arg2, ... ],
    method3 => [ $object, $method, $arg1, $arg2, ... ],
    method4 => {
        class  => 'My::Delegate::Class',
        method => 'method_name',
        args   => [ $arg1, $arg2, ... ],
    },
    method5 => {
        object => $object,
        method => 'method_name',
        args   => [ $arg1, $arg2, ... ],
    },
});

$facade->method1($more_args1, ...);
$facade->method2($more_args2, ...);
```

Использование <font color="#00aa00">Class::Facade</font> :

```perl
package My::Facade::One;
use base qw( Class::Facade );

package My::Facade::Two;
use base qw( Class::Facade );

package main;
my $one = My::Facade::One->new({ ... });
my $two = My::Facade::Two->new({ ... });
```

## Очень простой пример реализации паттерна Facade

Frequest.pm :

```perl
package Frequest;
use strict;

sub new {
  my $type=shift;
  my $self;
  $self = bless( {}, $type);
  return $self;
}
sub send_request {
  my $self = shift;
  print "Log: send request\n";
  return 1;
}

1;
```

Fresponse.pm :

```perl
package Fresponse;
use strict;

sub new {
  my $type=shift;
  my $self;
  $self = bless( {}, $type);
  return $self;
}
sub get_response {
  my $self = shift;
  print "Log: get response\n";
  return 1;
}

1;
```

Фасад - Facade.pm :

```perl
package Facade;

use strict;

use Frequest;
use Fresponse;

sub new {
  my $type=shift;
  my $self;
  $self = bless( {}, $type);
  return $self;
}
sub process {
  my $self = shift;
  Frequest->new->send_request;
  Fresponse->new->get_response;
  print "Process... done\n";
  return 1;
}

1;
```
Вместо вызова нескольких модулей, вызываем только модуль Facade.pm.

facade.pl :

```perl
#!/usr/bin/perl

use strict;
use Facade;

my $obj = Facade->new;
$obj->process();
exit;
```

