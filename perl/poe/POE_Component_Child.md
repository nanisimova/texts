# Компонент POE::Component::Child. Создание дочерних процессов в POE

## Простой пример использования POE::Component::Child

С помощью компоненты POE::Component::Child мы создаем POE-программу, которая для выполнения некоторых задач создает дочерние процессы.

Сначала создается и выполняется первый дочерний процесс, после завершения работы первого - второй. Когда и второй закончит свою работу - программа завершается.


```perl
#!/usr/bin/perl

use warnings;
use strict;

use POE;
use POE::Component::Child;

my $debug = 0;

# установка соответствия событий и их обработчиков
my %events = (
  stdout => "my_stdout",
  stderr => "my_stderr",
  error  => "my_error",
  done   => "my_done",
  died   => "my_died",
);

POE::Session->create(
  package_states => ["main" => [values(%events)]],
  inline_states  => {
    _start => sub { $_[KERNEL]->alias_set("main"); },
    _stop  => sub { print "_stop" if $debug; },
  }
);

my $c = POE::Component::Child->new(
  events => \%events,
  debug  => $debug
);

# запускаем первый дочерний процесс
$c->run("ls");

# запускаем второй дочерний процесс
$c->run("ps", "-f");

POE::Kernel->run();
exit;

sub my_stdout {
  my ($self, $args) = @_[ARG0 .. $#_];
  print $args->{out}, $/;
}

sub my_stderr {
  my ($self, $args) = @_[ARG0 .. $#_];
  print ">> $args->{out}";
}

sub my_done {
  print "child done\n";
}

sub my_died {
  print "child died!\n";
}

sub my_error {
  my ($self, $args) = @_[ARG0 .. $#_];
  print "$args->{error}\n";
}
```

Результат работы скрипта:
<pre>$ perl poe_child.pl
poe_child.pl
simple_ping.pl
simple_server.pl
child done
UID        PID  PPID  C STIME TTY          TIME CMD
1366     18169 30283  0 15:41 pts/12   00:00:00 perl poe_child.pl
1366     18173 18169  0 15:41 pts/12   00:00:00 ps -f
1366     30283 30281  0 Aug20 pts/12   00:00:00 bash
child done
</pre>


## Краткий обзор возможностей компонента POE::Component::Child

Компонент POE::Component::Child является оболочкой для <font color="#00aa00">POE::Wheel::Run</font>. Делает использование POE::Wheel::Run более удобным и предоставляет объектно-ориентированный интерфейс к нему. Дочерние процессы создаются непосредственно POE::Wheel::Run, с помощью <font color="#00aa00">fork</font>.

### Методы компонента

#### new

Метод создает экземпляр компонента. На вход принимает либо хеш, либо ссылку на хеш.
<ul>
 	<li><b>alias</b> - Задает имя сессии, по которому к ней можно будет обращаться при вызове <font color="#00aa00">post</font>. По умолчанию - <font color="#00aa00">main</font>.</li>
 	<li><b>events</b> - Ссылка на хеш, который содержит список событий для текущего компонента, со ссылками на соответствующие обработчики. Варианты допустимых событий: <font color="#00aa00">stdout</font>, <font color="#00aa00">stderr</font>, <font color="#00aa00">done</font>, <font color="#00aa00">died</font>, <font color="#00aa00">error</font>.</li>
 	<li><b>debug</b> - установка флага включает вывод отладочной информации.</li>
</ul>

#### run

Метод принимает список команд и параметров для запуска. Возвращает <font color="#00aa00">id wheel</font>, который может пригодиться, если запускается несколько разных команд.

#### write

Метод может использоваться для отправки массива информации дочернему процессу.

#### quit

Организует корректное завершение программ из дочерних сессий. Однако, полное закрытие дочерних процессов организуется с помощью метода <font color="#00aa00">shutdown()</font>.

#### kill

Посылает сигнал для насильственного завершения программ.

#### shutdown

Метод завершает работу всех дочерних процессов и закрывает их.

#### wheelid

Используется для получения <font color="#00aa00">wheelid</font>. В дальнейшем может использоваться для вызова методов, например, <font color="#00aa00">kill()</font> или <font color="#00aa00">shutdown()</font>.

### События дочерних процессов

Обработчики событий всегда получают на вход два аргумента: <font color="#00aa00">ARG0</font> и <font color="#00aa00">ARG1</font>. Первый содержит ссылку на объект, второй - хеш с параметрами.

Например, при выводе результатов в <font color="#00aa00">STDOUT</font>, функции-обработчику передавался в качестве <font color="#00aa00">ARG1</font> хеш, с ключами <font color="#00aa00">out</font> и <font color="#00aa00">wheel</font>. <font color="#00aa00">Out</font> содержал выводимую строку данных, <font color="#00aa00">wheel</font> - id.

#### stdout

Данное событие генерируется при любом выводе данных. Вывод данных осуществляется построчно. В нашем примере для вывода 3х-строк со списком процессов, вызов события <font color="#00aa00">stdout</font> и его обработчика <font color="#00aa00">my_stdout</font> осуществляется 3 раза.

#### stderr

Событие генерируется при любом выводе в <font color="#00aa00">STDERR</font>.

#### done

Генерируется после завершения работы дочернего процесса.

#### died

Вызывается в случае аварийного завершения дочернего процесса. Является взаимоисключающим с <font color="#00aa00">done</font>.

#### error

Событие генерируется, если дочерний процесс вернул ошибку.

