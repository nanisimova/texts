# Test::Harness

## Синтаксис

Функции модуля **Test::Harness** осуществляют запуск стандартных тестовых скриптов Perl с последующим выводом статистических отчетов. Модуль способен существенно облегчить управление большими наборами тестовых файлов.

<pre>
use Test::Harness;
runtests(@test_files);
</pre>

## Экспорт

Для работы с функцией runtests() достаточно просто подключить модуль Test::Harness. Функция экспортируется по умолчанию.

<pre>use Test::Harness;</pre>

Функция execute_tests, а так же переменные $verbose, $switches, $debug экспортируются по запросу.

<pre>use Test::Harness qw(execute_tests);</pre>


## Функции Test::Harness

### runtests( @test_files )

Функция запускает на выполнение все перечисленные в массиве @test_files тесты. Информацию о процессе тестирования и результатах получает из данных, которые направляются тестами в STDOUT. Выводит результаты об ошибках и информацию об успешно пройденных тестах в общем отчете. В случае успешного завершения всех тестов, возвращает true. В противном случае, завершает работу и возвращает диагностическое сообщение.

**Пример использования:**

```perl
#!/usr/bin/perl

use Test::Harness;

@test_files = qw(test_more.pl test_simple.pl);
runtests(@test_files);
```

**Вывод результатов работы функции:**

<pre>
%perl test_harness.pl
test_more......ok
test_simple....NOK 1
#   Failed test '1+1=3'
#   in test_simple.pl at line 5.
test_simple....ok 2/2# Looks like you failed 1 test of 2.
test_simple....dubious
        Test returned status 1 (wstat 256, 0x100)
DIED. FAILED test 1
        Failed 1/2 tests, 50.00% okay
Failed Test    Stat Wstat Total Fail  List of Failed
-------------------------------------------------------------------------------
test_simple.pl    1   256     2    1  1
Failed 1/2 test scripts. 1/3 subtests failed.
Files=2, Tests=3,  0 wallclock secs ( 0.05 cusr +  0.01 csys =  0.06 CPU)
Failed 1/2 test programs. 1/3 subtests failed.
</pre>


### execute_tests( tests =&gt; \@test_files, out =&gt; \*FH )

Подобно runtests(), функция execute_tests запускает на выполнение все заданные в массиве @test_files тесты, но при этом не генерирует общий итоговый отчет. В процессе тестирования, вся информация выводится по указанному пути (по-умолчанию, в STDOUT).

**Пример:**

```perl
#!/usr/bin/perl

use Test::Harness qw(execute_tests);

@test_files = qw(test_more.pl test_simple.pl);
execute_tests(tests => \@test_files);
```

**Вывод:**

<pre>
%perl test_harness.pl
test_more......ok
test_simple....NOK 1
   Failed test '1+1=3'
   in test_simple.pl at line 5.
test_simple....ok 2/2# Looks like you failed 1 test of 2.
test_simple....dubious
        Test returned status 1 (wstat 256, 0x100)
DIED. FAILED test 1
        Failed 1/2 tests, 50.00% okay
</pre>

Кроме общего вывода, функция execute_tests возвращает список из двух значений **$total** и **$failed**. **$total** - это
ссылка на хэш, который содержит суммарную информацию о всех пройденных тестах. Его стандартные ключи и значения:

<pre>
bonus		Number of individual todo tests unexpectedly passed
max		Number of individual tests ran
ok		Number of individual tests passed
sub_skipped	Number of individual tests skipped
todo		Number of individual todo tests
files		Number of test files ran
good		Number of test files passed
bad		Number of test files failed
tests		Number of test files originally given
skipped		Number of test files skipped
</pre>

**$failed** - это ссылка на хэш, который содержит информацию о тестах, которые при прохождении потерпели неудачу.
Ключами хэша являются названия файлов тестов, а значения - информация о причинах неудачного тестирования.

<pre>
name	Name of the test which failed
estat	Script's exit value
wstat	Script's wait status
max	Number of individual tests
failed	Number which failed
canon	List of tests which failed (as string).
</pre>

**Пример использования переменных $total и $failed:**

```perl
#!/usr/bin/perl

use Test::Harness qw(execute_tests);
@test_files = qw(test_more.pl test_simple.pl);
($total, $failed) = execute_tests(tests => \@test_files);

print "Total:\n";
foreach $key (keys %$total) {
        print $key." :  ". $total->{$key}."\n";
}
print "\nFailed:\n";
foreach $key (keys %$failed) {
        print $key." :  ". $failed->{$key}->{max}."\n";
}
```

**Вывод:**
<pre>
%perl test_harness.pl
test_more......ok
test_simple....NOK 1
   Failed test '1+1=3'
   in test_simple.pl at line 5.
 Looks like you failed 1 test of 2.
test_simple....dubious
        Test returned status 1 (wstat 256, 0x100)
DIED. FAILED test 1
        Failed 1/2 tests, 50.00% okay
Total:
files : 2
max :   3
bonus : 0
skipped :       0
sub_skipped :   0
ok :    2
bad :   1
good :  1
tests : 2
bench : Benchmark=ARRAY(0x81383bc)
todo :  0
Failed:
test_simple.pl :  2
</pre>

## Переменные окружения, которые устанавливает TEST::HARNESS

Данные переменные Test::Harness устанавливает перед запуском тестов на выполнение.
<ul>
<li>HARNESS_ACTIVE - Принимает истинное (true) значение. Переменная позволяет тестам определить, запущены они на выполнение с помощью Test::Harness, или каким-либо другим способом.</li>
<li>HARNESS_VERSION - Номер версии модуля Test::Harness.</li>
</ul>

## Переменные окружения, которые влияют на работу TEST::HARNESS

<ul>
<li>HARNESS_TIMER - присвоение этой переменной истинного значения (true), будет служить Test::Harness указанием выводить для каждого теста количество миллисекунд, в течение которых он выполнялся.</li>
<li>HARNESS_VERBOSE - если переменной задано истинное (true) значение, Test::Harness будет выводить отчеты с большими текстами описаний. Для установки значений можно использовать переменную $Test::Harness::verbose. Значение $Test::Harness::verbose является приоритетным.</li>
<li>HARNESS_OPTIONS - дополнительные параметры. В настоящее время поддерживаются опции:
<ul>
 	<li>j&lt;n&gt; - запуск &lt;n&gt; (по умолчанию 9) параллельных процессов.</li>
 	<li>f - использование fork.</li>
</ul>
</li>
</ul>

Разделение нескольких параметров производится с помощью двоеточия.
<pre>HARNESS_OPTIONS=j9:f make test</pre>
