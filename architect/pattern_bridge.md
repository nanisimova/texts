# Шаблон проектирования Bridge

Назначение шаблона Bridge - отделить абстракцию от ее реализации так, чтобы и то, и другое можно было изменять независимо.

<img src="Bridgeuml.gif" alt="Bridgeuml" class="aligncenter size-full wp-image-1722" style="margin: 6px; border: 1px solid black; padding: 6px;" width="448" height="293">

Проще всего понять, что такое шаблон Bridge, если вспомнить компоненту View в модели MVC.

Допустим, у нас есть текстовые объекты различных типов: Книги, статьи, динамически-формируемые страницы каталогов, справки и т.п. При этом, каждый документ может быть представлен в разных форматах: pdf, rtf, html и др.

Если мы начнем создавать иерархии классов вот так:
<pre>
* Document
* Document::Blank
* Document::Blank::Html
* Document::Blank::PDF
* Document::Blank::XML
* Document::Book
* Document::Book::Html
* Document::Book::Fb2
...
</pre>
то модификация системы, добавление нового формата данных, или нового типа данных превращается в работу весьма сложную, чреватую дублированием кода, структура программы становится запутанной.

Именно поэтому, в данном случае будет полезно разделить саму структуру объектов и работу над их внешним видом.

У нас появляется новая структура:
<pre>
* Document
* Document::Blank
* Document::Book
* View
* View::Html
* View::PDF
* View::XML
* View::Fb2
</pre>

В ней присутствует деление на абстракцию Abstraction (объекты документов) и реализацию Implementation (компоненты View).

Такая система легко расширяется.

Указанные принципы структурирования программы используются и в тех случаях, когда необходимо сделать переносимый графический интерфейс для какой-либо программы - создаются классы, которые будут отвечать за отрисовку интерфейса в разных средах.

Почему этот шаблон называют "Bridge" - мне не понятно. Этот шаблон так же имеет название "Handle/Body". Мне кажется, это чуть ближе по смыслу.

## Пример реализации паттерна Bridge - 1

test.pl :

```perl
use strict;

use Document::Book;
use View::Html;
use View::PDF;

my $document = Document::Book->new(View::Html->new);
$document->create;
$document = Document::Book->new(View::PDF->new);
$document->create;
```

Document.pm :

```perl
package Document;

use strict;
use warnings;

sub new {
	my ($class, $imp) = @_;
	my $self = {
		imp => $imp,
	};
	return bless $self, $class;
}
sub create {
	my $self = shift;
	$self->{imp}->operateImp;
}
1;
```

Document/Book.pm :

```perl
package Document::Book;
use parent Document;

sub new {
	my ($class, $imp) = @_;
	my $self = $class->SUPER::new($imp);
	return $self;
}
sub operate {
	my $self = shift;
	$self->{imp}->operateImp;
}
1;
```

View.pm :

```perl
package View;

use strict;
use warnings;

sub new {
	my ($class, $args) = @_;
	my $self = {};
	return bless $self, $class;
}
sub operateImp {
	die "ABSTRACT CLASS METHOD CANNOT BE CALLED DIRECTLY\n";
}
1;
```

View/Html.pm :

```perl
package View::Html;
use parent View;

use strict;
use warnings;

sub new {
	my ($class, $args) = @_;
	my $self = $class->SUPER::new($args);
	return $self;
}
sub operateImp {
	my $self = shift;
	print "Print document in html format\n";
}

1;
```

View/PDF.pm :

```perl
package View::PDF;
use parent View;

use strict;
use warnings;

sub new {
	my ($class, $args) = @_;
	my $self = $class->SUPER::new($args);
	return $self;
}
sub operateImp {
	my $self = shift;
	print "Print document in pdf format\n";
}

1;
```

Запускаем скрипт на выполнение:

<pre>$ test.pl
Print document in html format
Print document in pdf format
</pre>

## Пример реализации паттерна Bridge - 2

Чистый пример, с соответствующей терминологией.

test.pl :

```perl
use strict;

use Abstraction::RefinedAbstraction;
use Implementator::ConcreteImplementatorA;
use Implementator::ConcreteImplementatorB;

my $abs = Abstraction::RefinedAbstraction->new(Implementator::ConcreteImplementatorA->new);
$abs->operate;
$abs = Abstraction::RefinedAbstraction->new(Implementator::ConcreteImplementatorB->new);
$abs->operate;
```

Abstraction.pm :

```perl
package Abstraction;

use strict;
use warnings;

sub new {
	my ($class, $imp) = @_;
	my $self = {
		imp => $imp,
	};
	return bless $self, $class;
}
sub implementator {
	my ($self, $imp) = @_;
	$self->{imp} = $imp if defined $imp;
	return $self->{imp};
}
sub operate {
	my $self = shift;
	$self->{imp}->operateImp;
}

1;
```

Abstraction/RefinedAbstraction.pm :

```perl
package Abstraction::RefinedAbstraction;
use parent Abstraction;

sub new {
	my ($class, $imp) = @_;
	my $self = $class->SUPER::new($imp);
	return $self;
}
sub operate {
	my $self = shift;
	print ref $self, " operate:\n";
	$self->{imp}->operateImp;
}

1;
```

Implementator.pm :

```perl
package Implementator;

use strict;
use warnings;

sub new {
	my ($class, $args) = @_;
	my $self = {};
	return bless $self, $class;
}
sub operateImp {
	die "ABSTRACT CLASS METHOD CANNOT BE CALLED DIRECTLY\n";
}

1;
```

Implementator/ConcreteImplementatorA.pm :

```perl
package Implementator::ConcreteImplementatorA;
use parent Implementator;

use strict;
use warnings;

sub new {
	my ($class, $args) = @_;
	my $self = $class->SUPER::new($args);
	return $self;
}
sub operateImp {
	my $self = shift;
	print ref $self, " operate Imp\n";
}

1;
```

Implementator/ConcreteImplementatorB.pm :

```perl
package Implementator::ConcreteImplementatorB;
use parent Implementator;

use strict;
use warnings;

sub new {
	my ($class, $args) = @_;
	my $self = $class->SUPER::new($args);
	return $self;
}
sub operateImp {
	my $self = shift;
	print ref $self, " operate Imp\n";
}

1;
```

Запускаем на выполнение:

<pre>
$ test.pl
Abstraction::RefinedAbstraction operate:
Implementator::ConcreteImplementatorA operate Imp
Abstraction::RefinedAbstraction operate:
Implementator::ConcreteImplementatorB operate Imp
</pre>
