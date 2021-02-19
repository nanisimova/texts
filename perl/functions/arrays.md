# Встроенные функции Perl. Работа с массивами

## Функция push. Добавление новых элементов в конец массива

<pre>push ARRAY, LIST</pre>
Функция добавляет в конец массива <font color="#00aa00">ARRAY</font> один или несколько новых элементов. Возвращает новое число элементов в массиве.

Пример:

```perl
my @array = ('Alisa', 'Arina', 'Lisa');
push @array, 'Maria', 'Arline';

my @array2 = ('Triniti','Nina');
push @array, @array2;
```

## Функция pop. Удаление последнего элемента массива

<pre>pop ARRAY</pre>
Функция <font color="#00aa00">pop</font> возвращает и удаляет последний элемент массива. Если в массиве нет элементов, функция вернет undef.

Пример:

```perl
my @array = ('Alisa', 'Arina', 'Lisa');
pop @array;

print join(' ', @array)."\n"; # вывод: Alisa Arina
```

## Функция unshift. Добавление новых элементов в начало массива

<pre>unshift ARRAY, LIST</pre>
Функция добавляет новые элементы в начало массива и возвращает новое число элементов в массиве.

Пример:

```perl
my @array = ('Alisa', 'Arina', 'Lisa');
my $num = unshift @array, 'Samanta';

print join(' ', @array)."\n"; # вывод: Samanta Alisa Arina Lisa
print $num."\n"; # вывод: 4
```

## Функция shift. Удаление первого элемента массива

<pre>shift ARRAY</pre>
Функция возвращает и удаляет первый элемент массива. Элемент удаляется полностью, остальные элементы "смещаются" ближе к началу массива.

Пример:

```perl
my @array = ('Alisa', 'Arina', 'Lisa');
my $name = shift @array;

print join(' ', @array)."\n"; # вывод: Arina Lisa
print $name."\n"; # вывод: Alisa
```
Если <font color="#00aa00">shift</font> вызывается без явного указания массива, то будет прочитан, возвращен и удален первый элемент массива @_. Именно это свойство используется при передаче значений подпрограммам.

```perl
check_params($name, \%adress_params);

sub check_params {
	$name = shift;
	$adress_params = shift;
	#...
}
```

## Функция splice

<pre>splice ARRAY, OFFSET, LENGHT, LIST</pre>

<font color="#00aa00">OFFSET, LENGHT, LIST</font> - необязательные параметры. Удаляет заданное количество (<font color="#00aa00">LENGHT</font>) элементов из массива, начиная с элемента <font color="#00aa00">OFFSET</font>, и заменяет их новыми элементами (<font color="#00aa00">LIST</font>).

<font color="#00aa00">OFFSET</font> может иметь отрицательное значение. В этом случае, счет элементов идет с конца массива.

Если <font color="#00aa00">LENGHT</font> не задана - функция удалит все элементы, начиная с элемента <font color="#00aa00">OFFSET</font> и до конца массива.

Фактически, может легко реализовать функциональность <font color="#00aa00">shift, unshift, pop</font> и <font color="#00aa00">push</font>.  Но для чтения и понимания кода - функция не удобна.

```perl
my @array = ('Alisa', 'Arina', 'Lisa');
splice(@array, 1, 0, 'Erika');

print join(' ', @array)."\n"; # вывод: Alisa Erika Arina Lisa
splice(@array); # удалит все элементы массива
```

В списковом контексте функция возвращает элементы, удаленные из массива. В скалярном контексте - число удаленных элементов.

