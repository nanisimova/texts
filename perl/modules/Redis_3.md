# Использование perl-модуля Redis.pm. Часть 3

## Методы для операций над списками в значениях ключей

<font color="#00aa00">Redis Lists</font> - это списки, содержащие строковые значения в порядке их добавления.

Новые значения можно добавлять в начало и конец списка.

По сути, <font color="#00aa00">Redis Lists</font> - это то же, что и массивы в <font color="#00aa00">Perl</font>.

### Метод rpush

<pre>$r-&gt;rpush( $key, $value );</pre>

Добавление элемента в конец списка. Если заданный ключ еще не существует, он будет создан с пустым списком в качестве значения.

```perl
my $status = $redis->rpush('store.mobile.Nokia.N8', 'MTC' );
```

### Метод lpush

<pre>$r-&gt;lpush( $key, $value );</pre>

Добавление элемента в начало списка.

```perl
$status = $redis->lpush('store.mobile.Nokia.N8', 'Euroset');
```

### Метод llen

<pre>$r-&gt;llen( $key );</pre>

Получение числа - количества элементов списка.

```perl
my $key = $redis->llen('store.mobile.Nokia.N8');
print $key."\n"; # в данном случае выведет 6
```

### Метод lrange

<pre>my @list = $r-&gt;lrange( $key, $start, $end );</pre>

Получение ряда элементов из списка.

<font color="#00aa00">$start</font> - порядковый номер элемента, с которого начинается выборка. Номер первого элемента в списке - 0. <font color="#00aa00">$end</font> - порядковый номер элемента, которым заканчивается выборка.

Можно задать <font color="#00aa00">$start</font> и <font color="#00aa00">$end</font> отрицательные значения. Тогда выборка элементов будет идти с конца списка. При этом, номер последнего элемента будет иметь значение -1. -2 - это предпоследний элемент, и т.д.

```perl
my @list = $redis->lrange('price.mobile.Nokia.N8', 0, 2);
print join(" ", @list)."\n";
```

### Метод ltrim

<pre>my $ok = $r-&gt;ltrim( $key, $start, $end );</pre>

Обрезка списка. <font color="#00aa00">$start</font> и <font color="#00aa00">$end</font> - порядковые номера элементов списка - означают границы обрезки. Будет обрезано все, что присутствовало в списке до элемента <font color="#00aa00">$start</font> и после элемента <font color="#00aa00">$end</font> .

Если для <font color="#00aa00">$start</font> и <font color="#00aa00">$end</font> заданы отрицательные значения, отсчет элементов будет идти с конца списка. При этом, номер последнего элемента будет иметь значение -1.

```perl
$status = $redis->ltrim('numbers', 0, 1 );
```

### Метод lindex

<pre>$r-&gt;lindex( $key, $index );</pre>

Получение элемента из указанной позиции.

```perl
my $list_value = $redis->lindex('numbers', 2 );
```

### Метод lset

<pre>$r-&gt;lset( $key, $index, $value );</pre>

Установка нового значения для элемента в указанной позиции.

```perl
$redis->lset('numbers', 2, '111');
```

### Метод lrem

<pre>my $modified_count = $r-&gt;lrem( $key, $count, $value );</pre>

Удаление элементов из списка. Удаляются элементы, значение которых равно <font color="#00aa00">$value</font>. Число удаляемых элементов задается <font color="#00aa00">$count</font>.

Удаление элементов идет с начала списка. Если <font color="#00aa00">$count</font> задано
отрицательное значение - удаление элементов будет идти с конца списка.

Если <font color="#00aa00">$count</font> имеет значение 0 - из списка будут удалены все элементы, значение которых равно <font color="#00aa00">$value</font>.

Метод возвращает число удаленных элементов.

```perl
# до lrem список содержал значения 13 12 111 111 112 111
my $modified_count = $redis->lrem('numbers', -1, '111');
# после lrem - 13 12 111 111 112
```

### Метод lpop

<pre>my $value = $r-&gt;lpop( $key );</pre>

Получение первого элемента списка и его удаление.

```perl
# содержимое массива до lpop: 13 12 11 111 112 111
my $value = $redis->lpop('numbers');
# содержимое массива после lpop: 12 11 111 112 111
```

### Метод rpop

<pre>my $value = $r-&gt;rpop( $key );</pre>

Получение последнего элемента списка и его удаление.

```perl
# содержимое массива до rpop: 12 11 111 112 111
my $value = $redis->lpop('numbers');
# содержимое массива после rpop: 12 11 111 112
```
