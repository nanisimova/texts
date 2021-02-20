# Что такое callback в perl?

## Что такое callback?

**Callback** - это ссылка на блок исполняемого кода, который передан в качестве аргумента другому коду - функции, процедуре и т.п.

Принцип структурирования программы через использование callbacks бывает полезен при организации систем виджетов, плагинов и т.п.

Callback - это тот случай, когда термин на русский язык лучше не переводить - проще будет понять принцип его работы.

В принципе, callback в perl не совсем то же, что и в С. И что важно - он значительно доступнее для понимания.

## Пример callback в perl

В данном примере мы получаем название команды и список параметров из командной строки.

В зависимости от того, какая команда была введена пользователем, упаковываем переданные в параметрах данные в тот или иной формат - в данном случае xml или html. Вместо того, чтобы писать списки для if-ов
<pre>
if ($command eq 'to_xml') {
} elsif ($command eq 'to_html') {
} elsif ($command eq 'to_tsv') {
} ... # ну сколько можно, этих форматов еще 120 штук
</pre>

мы создаем список подпрограмм-обработчиков и по запросу передаем ссылки на них:

```perl
#!/usr/bin/perl

use warnings;
use strict;

my $command;
my $params = [];

my $functions = {
  # упаковщик в xml
  'to_xml' => sub {
    my $params = shift;

    print qq{<!--?xml version="1.0" encoding="UTF-8"?-->\n};
    foreach (@$params) {
      print qq{<data>$_</data>\n};
    }
  },
  # упаковщик в html
  'to_html' => sub {
    my $params = shift;
    print qq{<title>Data</title>\n};
    foreach (@$params) {
      print qq{$_ \n};
    }
    print qq{};
  }
};

# разбираем содержимое командной строки
for my $arg ( @ARGV ) {
  if ( $arg =~ /^\s*--command=(.+)\s*$/ ) {
    $command = $1;
  } else { 
    push( @$params, $arg );
  }
};

sub execute {
  my $command = shift;
  my $params = shift;

  eval {
    $functions->{$command}->($params);
  };
  if ($@) {
    print "$@";
  }
}

execute($command, $params);
exit;
```

Вызов программы:
<pre>%perl callback.pl --command=to_html 1 34 3</pre>

## Второй пример callback в perl

Еще один пример. Существует некоторый список обработчиков, каждый их которых выполняет какую-то свою работу. В зависимости от ситуации, функция <font color="#00aa00">main()</font> получает произвольный список ссылок на те обработчики, которые надо запустить на выполнение.

В данном случае, список жестко прописан в массиве <font color="#00aa00">$functions</font>, но в реальной жизни он мог бы зависеть от переданных программе параметров, полученных данных, произошедших событий.

```perl
#!/usr/bin/perl

use warnings;
use strict;

my $callback_func_1 = sub {
	print "Handler 1 done\n";
};
my $callback_func_2 = sub {
	print "Handler 2 done\n";
};
my $callback_func_3 = sub {
	print "Handler 3 done\n";
};
my $functions = [$callback_func_1, $callback_func_3];
main($functions);

sub main {
	my $function_for_exec = shift;
	foreach (@$function_for_exec) {
		$_->();
	}
}
exit;
```
