# Использование perl-модуля Redis.pm. Часть 5

<font color="#00aa00">Redis Hash</font> - это неупорядоченный набор пар - "поле" и "значение".  И "поле", и "значение", являются строками.

Работа с этими хэшами мало чем отличается от работы с обычными Perl-хэшами.

## Методы для операций над хэшами в значениях ключей

### Метод hset

<pre>$r-&gt;hset( $key, $field, $value );</pre>

Устанавливает для поля <font color="#00aa00">$field</font> значение <font color="#00aa00">$value</font>. Если поле уже существует - значение будет обновлено, и метод <font color="#00aa00">hset</font> вернет значение "0". Если установлено новое поле - метод вернет "1".

Если указанного ключа <font color="#00aa00">$key</font> еще не существует - он будет создан.

<pre>my $status = $redis-&gt;hset('store.programm.telnet_client', 'ssh', 'yes' );</pre>

### Метод hget

<pre>$r-&gt;hget( $key, $field );</pre>

Получить значение для поля <font color="#00aa00">$field</font>.

<pre>$value = $redis-&gt;hget('store.programm.telnet_client', 'ssh' );</pre>

### Метод hmset

<pre>$r-&gt;hmset( $key, $field1, $value1, ..., $fieldN, $valueN,  );</pre>

То же, что и метод <font color="#00aa00">hset</font>, но устанавливает значения сразу для нескольких полей.

<pre>$status = $redis-&gt;hmset('store.programm.telnet_client', 'ssh', 'yes', 'telnet', 'yes' );</pre>

### Метод hmget

<pre>$r-&gt;hmget( $key, field1, $fieldN );</pre>

То же, что и метод <font color="#00aa00">hget</font>, но возвращает значения для нескольких заданных полей.

```perl
my @list = $redis->hmget('store.programm.telnet_client', 'ssh', 'telnet' );
print join(" ", @list)."\n"; # yes yes
```

### Метод hexists

<pre>$r-&gt;hexists( $key, $field );</pre>

Проверяет поле <font color="#00aa00">$field</font> на наличие в хэше. Возвращает "1" или "0".

<pre>$status = $redis-&gt;hexists('store.programm.telnet_client', 'color' );</pre>

### Метод hdel

<pre>$r-&gt;hdel( $key, $field );</pre>

Удаляет заданное поле их хэша. Возвращает "1" или "0".

<pre>$status = $redis-&gt;hdel('store.programm.telnet_client', 'color' );</pre>

### Метод hlen
<pre>$r-&gt;hlen( $key );</pre>

Возвращает общее число элементов (пар) хэша.

<pre>$lenght = $redis-&gt;hlen('store.programm.telnet_client');</pre>

### Метод hkeys

<pre>$r-&gt;hkeys( $key );</pre>

Возвращает список всех полей хэша.

```perl
@list = $redis->hkeys('store.programm.telnet_client');
print join(" ", @list)."\n"; # ssh telnet color
```

### Метод hvals

<pre>$r-&gt;hvals( $key );</pre>

Возвращает список значений всех полей хэша.

```perl
@list = $redis->hvals('store.programm.telnet_client');
print join(" ", @list)."\n"; # yes yes 256
```

### Метод hgetall

<pre>$r-&gt;hgetall( $key );</pre>

Возвращает все поля и значения хэша.

```perl
my @list = $redis->hgetall('store.programm.telnet_client');
print join(" ", @list)."\n"; 
# ssh yes telnet yes color 256
```

Или так.

```perl
my $status = $redis->hset('store.programm.telnet_client', 'ssh', 'yes' );
$status=$redis->hmset('store.programm.telnet_client', 'telnet', 'yes', 'color', '256');

my %list = $redis->hgetall('store.programm.telnet_client');
print join(" ", keys %list)."\n"; # ssh telnet color
print join(" ", values %list)."\n"; # yes yes 256
```

