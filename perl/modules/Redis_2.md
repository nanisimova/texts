# Использование perl-модуля Redis.pm. Часть 2

## Методы для операций над строковыми значениями ключей

### Метод set

Добавление нового ключа, со строковым значением.

```perl
my $status = $redis->set( 'price.mobile.Nok.N8' => '15.580' );
print $status."\n"; # выведет OK
```

Имена для ключей лучше давать, придерживаясь определенной системы, например, <font color="#00aa00">"type:id:field"</font> или <font color="#00aa00">"type.id.field"</font> . Это облегчит Вам работу, снизит умственные затраты на понимание программного кода и содержимого БД, расширит возможности поиска и сортировки данных.

### Метод get

Получение значения для заданного ключа.

```perl
my $value = $redis->get('price.mobile.Nok.N8');
print $value."\n"; # выведет 15.580
```

### Метод mget

Получение значений сразу для нескольких ключей.

```perl
@values = $redis->mget('price.mobile.Nok.6700_Classic', 'price.mobile.Nok.N8');
print join(', ', @values)."\n"; # выведет ", 15.580", т.к. первый ключ в базе отсутствует
```

## Методы для операций над множествами в значениях ключей

<font color="#00aa00">Redis Sets</font> - это неупорядоченные множества, состоящие из строк.

<font color="#00aa00">Redis</font> позволяет добавлять новые элементы к множеству, удалять старые, проверять элементы на наличие в множестве.

Значения элементов множества не могут повторяться. Если равнозначный элемент уже добавлен ко множеству - новый добавляться не будет, и функция <font color="#00aa00">SADD</font> вернет 0.

### Метод sadd

<pre>$r-&gt;sadd( $key, $member );</pre>

Добавление нового элемента ко множеству.

```perl
my $status =$redis->sadd('store.mobile.Nok.N8', 'StartMaster' );
```

Если равнозначный элемент уже присутствует во множестве, sadd вернет 0. Если указанный ключ еще не существует в БД, он будет создан.

### Метод srem

<pre>$r-&gt;srem( $key, $member );</pre>

Удаление заданного элемента множества.

```perl
$status = $redis->srem('store.mobile.Nok.N8', 'MTC');
```

### Метод scard

<pre>my $elements = $r-&gt;scard( $key );</pre>

Получение числа - количества элементов множества.

```perl
my $elements = $redis->scard('store.mobile.Nok.N8');
print "found members - ".$elements."\n"; # вывод: 11
```

### Метод sismember

<pre>$r-&gt;sismember( $key, $member );</pre>

Проверка элемента на наличие во множестве.

```perl
$status = $redis->sismember('store.mobile.Nok.N8', 'Ozoom');
```

### Метод sinter

<pre>$r-&gt;sinter( $key1, $key2, ... );</pre>

Вычисление пересечений между заданными множествами.

```perl
my $status = $redis->sadd('store.mobile.Nok.N8', 'Eurosets' );
$status = $redis->sadd('store.mobile.Nokia.N8', 'MTC' );
$status = $redis->sadd('store.mobile.Nok.6700_Classic', 'Eurosets' );
$status = $redis->sadd('store.mobile.Nok.6700_Classic', 'Ozoom' );
my @res = $redis->sinter('store.mobile.Nok.6700_Classic', 'store.mobile.Nok.N8');
# Массив @res содержит 1 элемент - 'Eurosets'
```
