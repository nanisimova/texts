# Test::More для начинающих

## Используемая терминология

Framework - это простая концептуальная структура, используемая для решения сложной проблемной задачи. В контексте программирования - framework это каркас программной системы, может включать вспомогательные программы, библиотеки кода, язык сценариев и другое ПО, облегчающее разработку и объединение разных компонентов большого программного проекта.

## Руководство Test::More

Руководство написано с использованием публикации Test::More. By Michael G Schwern. Copyright 2001-2002, 2004-2006 и является ее частичным переводом. Все примеры созданы автором специально для данного руководства.

Модуль Test::More поставляется со стандартным дистрибутивом Perl начиная с версии 5.8. Модуль для более ранней версии можно взять в CPAN. Модуль работает со всеми версиями Perl, начиная с версии 5.6.

Test::More - это полноценный фреймворк (framework) для написания тестовых скриптов. Имеет множество специализированных функций.

Синтаксис
<pre>
  use Test::More tests => 23;
  # or
  use Test::More qw(no_plan);
  # or
  use Test::More skip_all => $reason;

  BEGIN { use_ok( 'Some::Module' ); }
  require_ok( 'Some::Module' );

  # Various ways to say "ok"
  ok($got eq $expected, $test_name);

  is  ($got, $expected, $test_name);
  isnt($got, $expected, $test_name);

  # Rather than print STDERR "# here's what went wrong\n"
  diag("here's what went wrong");

  like  ($got, qr/expected/, $test_name);
  unlike($got, qr/expected/, $test_name);

  cmp_ok($got, '==', $expected, $test_name);

  is_deeply($got_complex_structure, $expected_complex_structure, $test_name);

  SKIP: {
      skip $why, $how_many unless $have_some_feature;

      ok( foo(),       $test_name );
      is( foo(42), 23, $test_name );
  };

  TODO: {
      local $TODO = $why;

      ok( foo(),       $test_name );
      is( foo(42), 23, $test_name );
  };

  can_ok($module, @methods);
  isa_ok($object, $class);

  pass($test_name);
  fail($test_name);

  BAIL_OUT($why);

  # UNIMPLEMENTED!!!
  my @status = Test::More::status;
</pre>


## Использование основных функций

### ok

<pre>ok($got eq $expected, $test_name);</pre>

Функция просто проверяет переданное ей выражение ($got eq $expected) на истинность. Если выражение истинно - тест пройден, если выражение ошибочно - тест не пройден.

$test_name - это очень короткое описание текущего теста, которое будет выведено на экран рядом с результатами теста. Использование $test_name не является обязательным, но т.к. работа с исходным кодом теста, благодаря ему, существеннно облегчается, лучше его использовать.

**Пример:**

<pre>ok(1+1 == 2,'1+1=2');</pre>

Вывод:
<pre>ok 1 - 1+1=2</pre>


### is, isnt

<pre>
is  ( $got, $expected, $test_name );
isnt( $got, $expected, $test_name );
</pre>

Подобно ok(), is() и isnt(), получают два аргумента и сравнивают их между собой с помощью "eq" и "ne", соответственно. Если сравнение прошло успешно, то тест считается пройденным.

В чем преимущество is() и isnt() перед ok()? is() и isnt() в случае неуспешного прохождения теста, предоставляют более подробный отчет о произошедших ошибках.

**Пример:**

```perl
#!/usr/bin/perl

use Test::More tests => 1;

$name = 'marina';
is( $name, 'svetlana',   'Return name can be "Svetlana"' );
```

Первый аргумент is() - это переменная для сравнения, второй аргумент - то, чем должна являться указанная переменная. Если $name не соответствует эталону, тест будет признан не успешным.

Вывод:
<pre>
%perl test_more.pl
1..1
not ok 1 - Return name can be "Svetlana"
   Failed test 'Return name can be "Svetlana"'
   in test_more.pl at line 6.
          got: 'marina'
     expected: 'svetlana'
 Looks like you failed 1 test of 1.
</pre>

### like

<pre>like( $got, qr/expected/, $test_name );</pre>

like() проводит сравнение первого аргумента (в данном случае, переменной $got) со вторым, которое является регулярным выражением.

**Пример:**

```perl
#!/usr/bin/perl

use Test::More tests => 3;
@arr = (
        {
        name => 'marina',
        test_name => 'get name from MODULE0',
        }, {
        name => 'svetlana',
        test_name => 'get name from MODULE1',
        }, {
        name => 'arina',
        test_name => 'get name from MODULE2',
        },
);

foreach $test (@arr){
        like($test->{name}, qr/arina$/, $test->{test_name});
}
```

Вывод:
<pre>
%perl test_more.pl
1..3
ok 1 - get name from MODULE0
not ok 2 - get name from MODULE1
   Failed test 'get name from MODULE1'
   in test_more.pl at line 21.
                   'svetlana'
     doesn't match '(?-xism:arina$)'
ok 3 - get name from MODULE2
 Looks like you failed 1 test of 3.
%</pre>

### unlike

<pre>unlike( $got, qr/expected/, $test_name );</pre>

Действия unlike() противоположны действиям like(). unlike() получает первый аргумент, сравнивает его со вторым, который является регулярным выражением. Но тест признается успешно пройденным, если соответствия между аргументами (в отличие от like() ), НЕ найдено.

### cmp_ok

<pre>cmp_ok( $got, $op, $expected, $test_name );</pre>

Функция представляет собой нечто среднее, между ok() и is(). Она позволяет сравнивать два полученных аргумента, самостоятельно задавая оператор для сравнения. При этом, можно использовать любые бинарные операторы perl.

**Пример:**

```perl
#!/usr/bin/perl

use Test::More tests => 3;

$page_size = 11;

cmp_ok( $page_size, '>', '10', 'get page_size' );
```

Вывод:
<pre>
%perl test_more.pl
1..3
ok 1 - get page_size
Looks like you planned 3 tests but only ran 1.
</pre>

### pass, fail

<pre>pass($test_name);
fail($test_name);</pre>

Используется, просто для вывода сообщения пользователю, что тест пройден (pass) или не пройден (fail). Применять данные функции можно в случаях, когда процедура теста была написана без использования функций Test::More.

**Пример:**

```perl
#!/usr/bin/perl

use PDF::API2;
use Test::More tests => 1;

pass('test1');
```
Вывод:
<pre>
%perl test_more.pl
1..1
ok 1 - test1
</pre>

## Тестирование объектно-ориентированного кода

### can_ok

<pre>can_ok($module, @methods);
can_ok($object, @methods);</pre>

Функция can_ok() позволяет проверить, есть ли у модуля ($module) или объекта ($object) возможность работы с методами (или функциями), указанными в списке @methods.

**Пример:**

```perl
#!/usr/bin/perl

use PDF::API2;
use Test::More tests => 2;

can_ok('PDF::API2', qw(new));

$pdf = PDF::API2->new;
can_ok($pdf, qw(page));
```

Вывод:
<pre>%perl test_more.pl
1..2
ok 1 - PDF::API2->can('new')
ok 2 - PDF::API2->can('page')
</pre>

Независимо от того, сколько функций (методов) будет перечислено в массиве @methods, выполнение can_ok() будет засчитано как выполнение ОДНОГО теста.

Если Вы хотите, чтобы проверка каждого метода (функции) проходила как отдельный независимый тест, следует использовать подобную конструкцию:
```perl
    foreach my $meth (@methods) {
        can_ok('Foo', $meth);
    }
```

### isa_ok

<pre>isa_ok($object, $class, $object_name);
isa_ok($ref,    $type,  $ref_name);</pre>

Проверяет, был ли создан объект и принадлежит ли он искомому классу (если не принадлежит, функция сообщит истинный класс объекта).

**Пример:**

```perl
#!/usr/bin/perl

use Test::More qw( no_plan );

use_ok('ExtUtils::Installed');

my $installed = ExtUtils::Installed->new();
isa_ok($installed, "ExtUtils::Install");
```

Вывод:
<pre>"packlist.t" 7 lines, 184 characters
%perl packlist.t
ok 1 - use ExtUtils::Installed;
not ok 2 - The object isa ExtUtils::Install
   Failed test 'The object isa ExtUtils::Install'
   in packlist.t at line 7.
     The object isn't a 'ExtUtils::Install' it's a 'ExtUtils::Installed'
1..2
 Looks like you failed 1 test of 2.
</pre>

## Тестирование модулей

### use_ok

<pre>use_ok($module, @imports);</pre>

Функция позволяет удостовериться, что подключение модуля c помощью команды use проходит успешно.

Помимо теста, функция автоматически подключает к тесту модуль, делая его доступным для использования. Т.е. выполняет работу директивы use:
<pre>use PDF::API2 qw(new open page);</pre>

Можно дополнительно задать массив @imports, который будет содержать список функций, которые мы хотели бы из указанного модуля импортировать. В этом случае будет проведена проверка возможности импорта для каждой из указанных функций.

**Пример:**

```perl
#!/usr/bin/perl

use PDF::API2;
use Test::More tests => 1;

use_ok('PDF::API2', qw(new open page));
```

Вывод:
<pre>%perl test_more.pl
1..1
ok 1 - use PDF::API2;</pre>

Разница между использованием use_ok и use состоит в том, что если с подключением модуля возникнут проблемы, при использовании use_ok работа тестового сценария не будет прервана, и все тесты, за исключением тех, для работы которых необходим подключаемый модуль (они заранее обречены на неудачу в данном случае), будут выполнены.

### require_ok

<pre>require_ok($module);
require_ok($file);</pre>

Смысл работы require_ok() аналогичен use_ok(). За исключением того, что проверяется возможность подключения дополнительных модулей (или файлов) не с помощью use, а через require.

## Тестирование сложных структур данных

### is_deeply

Вышеприведенных функций может оказаться недостаточно, если возникнет необходимость сравнить достаточно сложные структуры данных. Для сравнения 2х сложных структур данных можно использовать функцию is_deeply().

<pre>is_deeply( $got, $expected, $test_name );</pre>

Синтаксис функции is_deeply() похож на is(), за исключением того, что $got и $expected должны являться ссылками на структуры данных. is_deeply () будет сравнивать содержимое структур. Если в процессе сравнения будут найдены отличия - пользователь получит сообщение и указание места, где начинаются различия.

**Пример:**

```perl
#!/usr/bin/perl

use PDF::API2;
use Test::More tests => 1;

$got = {
        women => {
                names => ('Maria', 'Sophi'),
                colledge => 'St. Anna women school'
        },
        men => {
                names => ('Mark','Anatoly'),
        }
};

$expected = {
        women => {
                names => ('Maria', 'Sophi'),
                colledge => 'St. Anna women school'
        },
        men => {
                names => ('Rhider','Anatoly'),
        }
};
is_deeply($got, $expected, 'test structures');
```

Вывод:
<pre>%perl test_more.pl
1..1
not ok 1 - test structures
   Failed test 'test structures'
   in test_more.pl at line 26.
     Structures begin differing at:
          $got->{men}{names} = 'Mark'
     $expected->{men}{names} = 'Rhider'
 Looks like you failed 1 test of 1.
</pre>

## Диагностика

Во время выполнения тестов, написанных с использованием Test::More, пользователь получает информацию об ошибках. В большинстве случаев, этих сообщений будет достаточно для поиска ошибок. Но при необходимости, можно дополнить стандартные диагностические сообщения, своими текстами.

### diag

<pre>diag(@diagnostic_message);</pre>

@diagnostic_message - массив текстовых сообщений, который будет выведен в дополнение к стандартным диагностическим сообщениям в указанной вами ситуации.

**Пример:**

<pre>is_deeply($got, $expected, 'test structures') or diag("We got wrong structure from MODULE");</pre>

Вывод:
<pre>not ok 1 - test structures
   Failed test 'test structures'
   in test_more.pl at line 26.
     Structures begin differing at:
          $got->{men}{names} = 'Mark'
     $expected->{men}{names} = 'Rhider'
 We got wrong structure from MODULE
 Looks like you failed 1 test of 1.
</pre>

## Условное тестирование

Если тесты пишутся раньше, чем программный код, то они изначально обречены на неудачу. Существует ряд ситуаций, когда мы заранее знаем, что тест потерпит неудачу, и не обращаем на это внимание.

Модуль Test::More предусматривает такой вариант развития событий и позволяет пометить некоторые тесты, как неготовые к использованию (TODO). Это делает возможным добавление в тесты блоков программного кода для тестирования функциональных возможностей, которые в тестируемой программе пока отсутствуют. Во время тестирования подобные тесты будут помечены как непройденные или пропущенные.

Использование TODO позволяет избежать возникновения экстренных ситуаций, аварийного завершения работы тестирующей программы, при тестировании временного отсутствующих (или незавершенных) модулей, функций или подпрограмм.

### TODO: BLOCK

<pre>TODO: {
local $TODO = $why if $condition;
    # normal testing code goes here
}
</pre>

Объявляет блок тестов, выполнение которых может завершиться неудачей (возможно потому, что вы еще не закончили разработку нужных функций и модулей, или знаете об ошибке, но не успели ее исправить). $why - содержит строку-объяснение, почему тесты могут быть не выполнены. $condition - условие, при котором будет выводиться строка $why, необязательный параметр.

**Пример:**

```perl
#!/usr/bin/perl

use PDF::API2;
use Test::More tests => 2;

$test_param = 1;
TODO: {
        local $TODO = "module for PDF not finished" if $test_param == 1;

        use_ok('PDF::API2', qw(new open page));
        can_ok('PDF::API2', qw(new));
}
```

Вывод:
<pre>%perl test_more.pl
1..2
ok 1 - use PDF::API2; # TODO module for PDF not finished
ok 2 - PDF::API2->can('new') # TODO module for PDF not finished
</pre>

### SKIP: BLOCK

В некоторых ситуациях тесты приходится пропускать. Например, некоторые функциональные возможности тестируемого модуля могут быть доступны только при условии использования Perl определенной версии, или определенной операционной системы, или при наличии определенных модулей.

Для пропуска нужно пометить выбранные тесты как блок SKIP. В процессе тестирования модуль Test::More не станет исполнять помеченный SKIP тест, в отличие от блоков TODO, которые выполняются в любом случае. В начале блока вызывается функция skip(), которая указывает причину, по которой производится пропуск тестов и их количество.

Рекомендуется использовать SKIP как можно реже, в виду их не совсем правильной работы. Лучше, когда это возможно, использовать блоки TODO. Тесты SKIP лучше использовать только в исключительных случаях, когда они имеют характер необязательных.

<pre>SKIP: {
	skip $why, $how_many if $condition;
	# далее следует группа тестов
}</pre>

С помощью SKIP: мы объявляем блок тестов, выполнение которых при определенных условиях должно быть пропущено. $why - причина, по которой выполнение тестов не происходит, $how_many - сколько тестов не выполнено, $condition - условие, которое определяет выполнение тестов.

**Пример:**

```perl
#!/usr/bin/perl

use PDF::API2;
use Test::More tests => 2;

$using_pdf = 0;

SKIP: {
        skip "Problem with MODULE using", 2 if $using_pdf == 0;
        # далее следует группа тестов
        use_ok('PDF::API2', qw(new open page));
        can_ok('PDF::API2', qw(new));
}
```

Вывод:
<pre>%perl test_more.pl
1..2
ok 1 # skip Problem with MODULE using
ok 2 # skip Problem with MODULE using
</pre>
Когда Test::More пропускает тест, он выводит сообщение ok специального вида, чтобы сохранить порядок нумерации тестов и заодно сообщить тестирующей программе о том, что произошло.

### todo_skip

<pre>TODO: {
	todo_skip $why, $how_many if $condition;
	#...normal testing code...
}
</pre>

Бывают ситуации, когда, несмотря на ограничение "сомнительных" тестов рамками блоков, "непроходимый" тест способен привести к зависанию всю программу тестирования.

В подобных случаях лучше полностью пропускать тесты, которые не будут успешно пройдены, не пытаясь их выполнить.

Синтаксис todo_skip аналогичен синтаксису skip. Все тесты, которые будут указаны в приведенном блоке, при запуске тестирования отмечаются как завершившиеся ошибкой.

**Пример:**

```code
#!/usr/bin/perl

use PDF::API2;
use Test::More tests => 2;

$test_param=1;

TODO: {
        todo_skip "module for PDF not finished", 2 if $test_param==1;

        use_ok('PDF::API2', qw(new open page));
        can_ok('PDF::API2', qw(new));
}
```

Вывод:
<pre>%perl test_more.pl
1..2
not ok 1 # TODO &amp; SKIP module for PDF not finished
not ok 2 # TODO &amp; SKIP module for PDF not finished
</pre>

## Дополнительные функции сравнения

Нижеприведенные функции в настоящее время используются все реже, т.к. практически не предоставляют диагностической информации об ошибках, и фактически даже не является тестом (их не нужно учитывать в строке use Test::More tests =&gt; 2;). Эти функции появились раньше, чем is_deeply(), который сейчас их фактически заменил.

Функции использовались преимущественно в связке с ok().
<pre>ok( eq_array(\@got, \@expected) );</pre>
is_deeply() имеет явные преимущества, предоставляя достаточно подробное описание происходящих ошибок и имея более лаконичную форму записи:

<pre>is_deeply( \@got, \@expected );</pre>
Вполне вероятно, что со временем, eq_array(), eq_hash() и eq_set() перестанут поддерживаться Test::More.

**eq_array**
<pre>my $is_eq = eq_array(\@got, \@expected);</pre>
Проводит сравнение массивов на идентичность содержания. Может проверять сложные, вложенные структуры массивов.

**Пример:**

```perl
#!/usr/bin/perl

use Test::More tests => 1;

@got = qw(Mary Tisha Miranda);
@expected = qw(Mary Miranda Aureliya);
ok(eq_array(\@got, \@expected));
```

Вывод:
<pre>
%perl test_more.pl
1..1
not ok 1
   Failed test in test_more.pl at line 8.
 Looks like you failed 1 test of 1.
</pre>

### eq_hash

<pre>my $is_eq = eq_hash(\%got, \%expected);</pre>
Проводит сравнение двух хэшей на полное соответствие. Может проверять вложенные структуры данных.

### eq_set

<pre>my $is_eq = eq_set(\@got, \@expected);</pre>
Сравнивает массивы, аналогично eq_array. Отличие состоит в том, что eq_set безразличен порядок, в котором следуют элементы массива.

**Пример:**

```perl
#!/usr/bin/perl

use Test::More tests => 1;

@got = qw(Aureliya Mary Miranda);
@expected = qw(Mary Miranda Aureliya);
ok(eq_set(\@got, \@expected));
```

Вывод:
<pre>%perl test_more.pl
1..1
ok 1
</pre>

Эту же задачу можно решить с помощью is_deeply():
<pre>is_deeply( [sort @got], [sort @expected] );</pre>

