# Что такое map в perl и как с ним работать

## Синтаксис

<pre>
map BLOCK LIST
map EXPR, LIST
</pre>

## Описание

Функция <font color="#00aa00">map</font> вычисляет BLOCK или EXPR для каждого элемента списка LIST, и возвращает список, который содержит преобразованные элементы LIST.

```perl
#!/usr/bin/perl -w

use strict;

my @list = qw(нет ни лета ни зимы ни весны);
my @names = map {split ''} @list;
foreach (@list) {print $_." "}; print "\n";
foreach (@names) {print $_." "}; print "\n";
```

Вывод:
<pre>
%perl map.pl
нет ни лета ни зимы ни весны
н е т н и л е т а н и з и м ы н и в е с н ы
</pre>

По приниципу работы <font color="#00aa00">map</font> напоминает <font color="#00aa00">foreach</font>. В случае сложных вычислений, использование <font color="#00aa00">foreach</font> может оказаться более предпочтительным, т.к. оно более наглядно и доступно для понимания.

От <font color="#00aa00">grep</font> функция <font color="#00aa00">map</font> отличается тем, что <font color="#00aa00">map</font> возвращает список, который является результатом вычислений в рамках EXPR или  BLOCK. А <font color="#00aa00">grep</font> возвращает список элементов LIST, которые соответствуют заданному условию поиска в EXPR или BLOCK.

Т.е. по сути, <font color="#00aa00">grep</font> используется в основном для поиска в заданном списке, а <font color="#00aa00">map</font> - для преобразований.

## Примеры использования map

### Хэш на выходе

```perl
my @list = qw(нет ни лета ни зимы ни весны);

my %names = map {$_ =&gt; 1} @list;
foreach (keys %names) {print $_." = ". $names{$_}."\n"};
```

Вывод:
<pre>
%perl map.pl
весны = 1
лета = 1
зимы = 1
ни = 1
нет = 1
</pre>

### Преобразование ссылок

```perl
#!/usr/bin/perl -w

use strict;

my @list = qw(<href="http://aninatalie.ru">MainPage);

map { s/\&amp;/&amp;/g;
          s/<!--&lt;/g;
          s/-->/&gt;/g;
          s/\"/"/g;
          s/\015//g;} @list;

foreach (@list) {print $_.""};
```

Вывод:
<pre>%perl map.pl
&amp;lt;href=&amp;quot;http://aninatalie.ru&amp;quot;&amp;gt;MainPage&amp;lt;/a&amp;gt;
</pre>
