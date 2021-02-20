# Шаблон проектирования Singleton

Данный шаблон гарантирует, что у класса есть только один экземпляр, и предоставляет к нему глобальную точку доступа.

Класс сам контролирует то, что у него есть только один экземпляр, может запретить создание дополнительных экземпляров, перехватывая запросы на создание новых объектов, и он же способен предоставить доступ к своему экземпляру. Это и есть назначение паттерна "одиночка".

## Пример реализации паттерна singleton для perl

Класс Singleton:

```perl
package Singleton;
 
use strict;
use vars qw($singleton);
 
sub new {
  my $type = shift;
  my $data = {};

  unless (defined $singleton) {
    $singleton = bless($data, $type);
  }
  return $singleton;
}
 
1;
```

Обычный класс, просто для контроля:

```perl
package Pack;
 
use strict;

sub new {
  my $type = shift;
  my $data = {};
  my $pack = bless($data, $type);
  return $pack;
}

1;
```

Скрипт, который использует оба перечисленных класса:

```perl
#!/usr/bin/perl

use Singleton;
use Pack;

my $a = Singleton->new;
my $b = Singleton->new;
print $a." ".$b."\n";

my $a1 = Pack->new;
my $b1 = Pack->new;
print $a1." ".$b1."\n";

my $c = Singleton->new;
print $c;
```

Результаты запуска скрипта:
<pre>
% perl singleton.pl
Singleton=HASH(0x9eaab4) Singleton=HASH(0x9eaab4)
Pack=HASH(0x9eab94) Pack=HASH(0x3f8bfc)
Singleton=HASH(0x9eaab4)
</pre>

Можно видеть, что при вызове метода new для класса singleton, в самом деле, объект создается только при первом вызове, а второй и третий раз - просто возвращается ссылка на уже существующий объект.

## Пример из жизни

Данный паттерн используется в POE, в коде ядра системы.

