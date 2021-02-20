# Использование perl-модуля Redis.pm. Часть 4

<font color="#00aa00">Redis ZSet</font> или <font color="#00aa00">Sorted Set</font> - это упорядоченное множество, коллекция строк. Каждый элемент такого множества, кроме собственно, значения-строки, имеет назначенную ему оценку (<font color="#00aa00">score</font>) . <font color="#00aa00">Score</font> служит для сортировки списка, и ее значение определяет положение элемента в списке. При изменении <font color="#00aa00">score</font> для элемента - меняется и положение этого элемента.


<font color="#00aa00">Score</font> должна быть строкой, представляющей собой число с плавающей точкой в двойной точности.

В целом, <font color="#00aa00">внешне</font>, упорядоченные множества напоминают Perl-массивы, с той разницей, что значение индекса можно задавать самостоятельно и совершенно произвольно. Можно создать список, который содержит всего два элемента, при этом первый элемент имеет <font color="#00aa00">score</font> - 5, а второй - 130.

## Методы для операций над упорядоченными множествами в значениях ключей

### Метод zadd

<pre>$r-&gt;zadd( $key, $score, $member );</pre>

Добавление ко множеству нового элемента с его оценкой.

```code
my $status = $redis->zadd('store.mobile.rating', 11, 'Smens' );
```

Если указанный ключ уже существует и содержит в себе значение другого типа, клиент получит
ошибку. Например:
<pre>[zadd] ERR Operation against a key holding the wrong kind of Siemens,  
at /usr/local/perl5/site_perl/5.8.8/Redis.pm line 277
</pre>

### Метод zrem

<pre>$r-&gt;zrem( $key, $member );</pre>

Удаление заданного элемента из множества.

```perl
$status = $redis->zrem('store.mobile.rating', 'Ssung');
print $status."\n";
```

### Метод zincrby

<pre>$r-&gt;zincrby( $key, $increment, $member );</pre>

Увеличение оценки (<font color="#00aa00">score</font>) заданного элемента на величину <font color="#00aa00">$increment</font>.

```perl
$status = $redis->zadd('store.mobile.rating', 34, 'Ssung' );

$status = $redis->zincrby('store.mobile.rating', 10, 'Ssung' );

my $score = $redis->zscore('store.mobile.rating', 'Ssung');
print $score."\n"; # выведет 44
```


### Метод zrank

<pre>$r-&gt;zrank( $key, $member );</pre>

Получение номера позиции элемента во множестве.

```perl
$position = $redis->zrank('store.mobile.rating', 'Ssung' );
print $position."\n"; # в данном случае выведет 1
```

### Метод zrevrank

<pre>$r-&gt;zrevrank( $key, $member );</pre>

Получение номера позиции элемента во множестве. В отличие от <font color="#00aa00">zrank</font>, поиск элемента начинается с другого конца множества.

```perl
my $status = $redis->zadd('store.mobile.rating', 11, 'Smens' );
$status = $redis->zadd('store.mobile.rating', 34, 'Ssung' );

$position = $redis->zrevrank('store.mobile.rating', 'Ssung');
print $position."\n"; # в данном случае выведет 0
```

### Метод zrange

<pre>$r-&gt;zrange( $key, $start, $end );</pre>

Получение набора отсортированных элементов множества, чьи позиции находятся в заданном диапазоне. <font color="#00aa00">$start</font> - номер позиции, с которой начинается выборка, <font color="#00aa00">$end</font> - номер позиции, которой она заканчивается.

```perl
my @list = $redis->zrange('store.mobile.rating', 0, 4);
print join(" ", @list)."\n"; # вывод: Smens Ssung
```


### Метод zrevrange

<pre>$r-&gt;zrevrange( $key, $start, $end );</pre>

Получение набора элементов, отсортированных в обратном порядке. <font color="#00aa00">$start</font> - номер позиции, с которой начинается выборка, <font color="#00aa00">$end</font> - номер позиции, которой она заканчивается.

```perl
my @list = $redis->zrevrange('store.mobile.rating', 0, 4);
print join(" ", @list)."\n"; # вывод: Ssung Smens
```


### Метод zrangebyscore

<pre>$r-&gt;zrangebyscore( $key, $min, $max );</pre>

Получение списка элементов с оценками из заданного диапазона. <font color="#00aa00">$min</font> - это минимальное значение <font color="#00aa00">score</font> в диапазоне, <font color="#00aa00">$max</font> - максимальное.

```perl
my @list = $redis->zrangebyscore('store.mobile.rating', 10, 40 );
```

### Метод zcount

<pre>$r-&gt;zcount( $key, $min, $max );</pre>

Получение количества элементов множества, чьи оценки (<font color="#00aa00">score</font>) находятся в заданном диапазоне. <font color="#00aa00">$min</font> - это минимальное значение <font color="#00aa00">score</font> в диапазоне, <font color="#00aa00">$max</font> - максимальное.

```perl
my $status = $redis->zadd('store.mobile.rating', 11, 'Smens' );
$status = $redis->zadd('store.mobile.rating', 34, 'Ssung' );

my $count = $redis->zcount('store.mobile.rating', 10, 40);
print $count."\n"; # в данном случае вернет 2
```

### Метод zcard

<pre>$r-&gt;zcard( $key );</pre>

Получить число - количество всех элементов множества.

```perl
my $status = $redis->zadd('store.mobile.rating', 11, 'Smens' );
$status = $redis->zadd('store.mobile.rating', 34, 'Ssung' );
my $count = $redis->zcard('store.mobile.rating');
print $count."\n"; # в данном случае выведет 2
```

### Метод zscore

<pre>$r-&gt;zscore( $key, $member );</pre>

Получить оценку (<font color="#00aa00">score</font>) для заданного элемента упорядоченного множества.

```perl
$status = $redis->zadd('store.mobile.rating', 34, 'Ssung' );
$score = $redis->zscore('store.mobile.rating', 'Ssung');
print $score."\n"; # выведет 34
```

