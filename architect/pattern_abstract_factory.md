# Шаблон проектирования Abstract Factory

<font color="#00aa00">Абстрактная фабрика</font> - паттерн, порождающий объекты. Известен также под именем <font color="#00aa00">Kit</font> (инструментарий).

<font color="#00aa00">Абстрактная фабрика</font> - это код, который решает, какой из множества подклассов должен использоваться в данный момент.

Исходный код запрашивает экземпляр класса. Данный класс возвращает экземпляр подкласса, который наиболее соответствует условиям. Подобный класс называют абстрактной фабрикой. Допустим, мы имеем дело с базой данных. В этом случае, абстрактная фабрика будет возвращать объект, который подходит для работы с конкретной базой данных. При этом, все подклассы имеют одинаковый API.

<font color="#00aa00">Perl DBI</font> - это отличный пример абстрактной фабрики. Каждый раз, при вызове <font color="#00aa00">connect</font>, фабрика ожидает, что ей будет передан тип базы данных, с которой предстоит работать. В зависимости от типа БД она будет подгружать <font color="#00aa00">DBD</font>. При этом, всегда можно добавить новые драйвера баз данных, и для начала их
использования не потребуется переписывать код системы.

Пример:

```perl
use DBI;
my $dbh = DBI->connect("dbi:mysql:dbname:localhost", "user", "password");
```

После подключения к БД и получения дескриптора <font color="#00aa00">$dbh</font>, можно обращаться к базе данных, не размышляя о том, какого она типа. Если вы решите переехать с <font color="#00aa00">mysql</font> на <font color="#00aa00">oracle</font>, придется изменить только название драйвера в строке <font color="#00aa00">connect</font>. Большая часть кода останется неизменной.

Кроме того, отличной демонстрацией работы <font color="#00aa00">Abstract Factory</font> являются системы виджетов. Все виджеты могут реализовывать разную функциональность. Один выводит форму для голосований, другой - рекламный баннер, и т.п. Для того, чтобы подключение нового виджета было быстрым и безболезненным, можно использовать шаблон <font color="#00aa00">Abstract Factory</font>.

Допустим, имеется список виджетов, которые требуется отобразить на сайте. Создаем абстрактный класс <font color="#00aa00">WidgetFactory</font>, который в зависимости от параметров, создает объект конкретного подкласса. Можно даже добавить типизацию виджетов. Тогда для каждого типа виджета можно создать собственную абстрактную фабрику, которая обращается к своим конкретным подклассам.

## Очень простой пример Abstract Factory

System/ASystem1.pm :

```perl
package System::ASystem1;

use strict;

sub new {
  my $type=shift;
  my $self;
  $self = bless( {}, $type);
  return $self;
}
sub process {
  my $self = shift;
  my $time = localtime();
  print "Make test\nShow time: $time\nProcess done\n";
  return 1;
}
1;
```

System/ASystem2.pm :

```perl
package System::ASystem2;

use strict;

sub new {
  my $type=shift;
  my $self;
  $self = bless( {}, $type);
  return $self;
}
sub process {
  my $self = shift;
  print "Process done\n";
  return 1;
}
1;
```

Фабрика имеет только один метод, который возвращает вызываемый тип объекта. Передаваемые данные она использует для определения нужного класса и модуля.

AbstractFactory.pm :

```perl
package AbstractFactory;

use strict;

sub create {
  my $class          = shift;
  my $requested_type = shift;
  my $location       = "System/$requested_type.pm";
  my $class          = "System::$requested_type";
  require $location;
  return $class->new(@_);
}
1;
```

В зависимости от требований разработчика, будет подключена та или иная система.

abstractfactory.pl :

```perl
#!/usr/bin/perl

use strict;
use AbstractFactory;

my $handler1 = AbstractFactory->create("ASystem1");
$handler1->process();
my $handler2 = AbstractFactory->create("ASystem2");
$handler2->process();

exit;
```
