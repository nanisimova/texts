# Что такое grep в perl и как его использовать

## Название

<font color="#00aa00">grep</font> - это встроенная в Perl функция, которая ищет в списке элементы, удовлетворяющие заданному условию. Используется для извлечения и преобразования данных.

Отличается от стандартной unix-программы <font color="#00aa00">grep</font> тем, что стандартный <font color="#00aa00">grep</font> выполняет поиск строк в одном или нескольких файлах, соответствующие заданному регулярному выражению.

## Синтаксис

```perl
grep EXPR, LIST
grep BLOCK LIST
```

## Описание

Функция вычисляет EXPR или BLOCK для каждого элемента LIST.

В списковом контексте возвращает элементы, для которых выражение EXPR или BLOCK является истинным. В скалярном контексте возвращает число - сколько элементов LIST соответствует заданному условию.

```perl
#!/usr/bin/perl -w

use strict;

my @list = qw(письма и телеграммы сгорели очень быстро и дотла
                были их килограммы теперь осталась только лишь зола);

my @result = grep /и/i, @list;
my $result_num = grep /и/i, @list;

foreach (@result) {print $_." "};

print "\n".$result_num."\n";
```

Вывод:
<pre>
%perl grep.pl
письма и сгорели и были их килограммы лишь
8
%
</pre>
Во время работы функция <font color="#00aa00">grep</font> поочередно присваивает переменной $_ значения элементов из списка, и проверяет это значение на соответствие заданному условию. Одновременно с отбором элементов, можно вносить в них изменения.

Главное при этом помнить, что EXPR или BLOCK могут выполнять модификацию исходного списка.

```perl
my @list = qw(письма и телеграммы сгорели очень быстро и дотла
                были их килограммы теперь осталась только лишь зола);

my @result = grep s/и/i/, @list;

foreach (@list) {print $_." "};
```

Вывод:
<pre>
%perl grep.pl
пiсьма i телеграммы сгорелi очень быстро i дотла былi iх кiлограммы
теперь осталась только лiшь зола
</pre>

## Примеры

### Поиск в сложной структуре данных

Есть список кандидатов на вакансию. Каждый кандидат обладает определенными навыками. Надо найти имена тех кандидатов, которые обладают списком заданных навыков (например: perl и mysql).

```perl
#!/usr/bin/perl -w

use strict;

my %persons = (
        'Мария' => [qw(html css mysql php perl xml apache)],
        'Сергей' => [qw(perl ruby apache nginx xml)],
        'Василий' => [qw(php html mysql)],
        'Захарий' => [qw(mysql oracle perl)],
);

my @names = grep {
                grep /mysql/, @{$persons{$_}} if grep /perl/, 
                @{$persons{$_}};
        } keys %persons;


foreach (@names) {print $_." "};
```

Вывод:
<pre>
Захарий Мария
</pre>

### Поиск в сложной структуре данных - 2
Требуется вывести имена сотрудников, специализирующихся на работе с oracle.

```perl
my @dbs = (
        {
                'name' => 'Natalie Ani',
                'specialization' => 'perl',
                'phone' => '4633',
        },
        {
                'name' => 'Mary Smirnova',
                'specialization' => 'ruby',
                'phone' => '4323',
        },
        {
                'name' => 'Alex Kravchik',
                'specialization' => 'oracle',
                'phone' => '4566',
        },
        {
                'name' => 'Alex Orlikov',
                'specialization' => 'oracle',
                'phone' => '4567',
        },
);

my @res = grep {
        $dbs[$_]->{'specialization'} eq 'oracle'
} 0..$#dbs;

foreach (@res) {print $dbs[$_]->{'name'}." "};
```

Вывод:
<pre>
%perl grep.pl
Alex Kravchik Alex Orlikov
</pre>

### Поиск одновременно в нескольких массивах

```perl
my @names = grep /^А/, (@women_names, @men_names);
```

### Поиск элементов по численному значению

```perl
my @num = qw(12 13 500 76 27 262 1 5 150 89 8);

my @res = grep {$_ &lt; 100} @num;
```

