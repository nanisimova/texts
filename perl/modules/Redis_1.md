# Использование perl-модуля Redis.pm для работы с БД Redis. Часть 1

<b>Краткая справка</b>

<font color="#00aa00">Redis</font> - это нереляционная база данных.

Используется в проектах с большими объемами информации и необходимостью быстрого доступа к ней.

Для хранения информации в <font color="#00aa00">Redis</font> используются пары - ключ и значение. Ключ должен быть простой строкой, без пробелов и символов переноса строки. Значение ключа может иметь тип данных: строка, список, не упорядоченное множество, упорядоченное множество или хэш.

Работать с <font color="#00aa00">Redis</font> значительно проще, чем с реляционными БД. Основной алгоритм работы: установить соединение с БД, получить значения определенных ключей, установить значения для ключей, закрыть соединение. И никаких вам INNER JOIN. Все возможные сортировки, отбор информации, осуществляется самим клиентом после получения данных.

Для работы с <font color="#00aa00">Redis</font> в Perl, существует специальный модуль - <font color="#00aa00">Redis.pm</font>.

## Методы модуля Redis.pm

Краткий обзор возможностей perl-модуля <font color="#00aa00">Redis.pm</font> для работы с <font color="#00aa00">Redis</font> БД.

### Подключение к БД

<ul>
<li>Метод <font color="#00aa00">new</font> - создание объекта - соединения с БД</li>
<li>Метод <font color="#00aa00">ping</font> - проверка соединения, готовности БД к работе</li>
<li>Метод <font color="#00aa00">info</font> - получение информации о сервере <font color="#00aa00">Redis</font></li>
<li>Метод <font color="#00aa00">quit</font> - разрыв соединения с БД</li>
</ul>

Пример:

```perl
#!/usr/local/bin/perl

use Redis;

my $redis = Redis->new(
        server => 'localhost:6379',
        encoding => undef,
);
$redis->ping || die "redis do not answer";
my $redis_info_hash = $redis->info;
foreach (keys %{$redis_info_hash}) {
        print $_.": ".$redis_info_hash->{$_}."\n";
}
$redis->quit;
```

Ответ:
<pre>
%perl redis.pl
last_save_time: 1302160813
bgsave_in_progress: 0
vm_enabled: 0
uptime_in_seconds: 3015123
pubsub_channels: 0
hash_max_zipmap_entries: 64
total_connections_received: 85496
redis_version: 2.0.4
process_id: 69098
total_commands_processed: 818953
hash_max_zipmap_value: 512
redis_git_dirty: 0
used_memory: 3562000
blocked_clients: 0
connected_clients: 1
#...
</pre>
&nbsp;

### Команды, применимые к любому ключу

#### Метод del

Удаление ключа из БД со всеми его значениями

```perl
my $status = $redis->del('price.mobile.Nokia.N8');
print $status."\n";
```

#### Метод exists

Проверка существования ключа

```perl
my $status = $redis->exists('price.mobile.Nokia.N8');
print $status."\n"; # вывод: 1 или 0 - если заданный ключ отсутствует 
```

#### Метод dbsize

Получение общего числа ключей, существующих в данной БД

```perl
my $keys = $redis->dbsize;
print $keys."\n"; # вывод: 44187
```

#### Метод randomkey

Получение случайного ключа из существующих.

```perl
my $key = $redis->randomkey;
print $key; # вывод: price.mobile.Nokia.6700_Classic
```

#### Метод type

Получение типа значения, сохранённого в ключе.

```perl
my $key = $redis->type('price.mobile.Nokia.N8');
print $key."\n"; # вывод: string
```

#### Метод rename

Переименование ключа.

```perl
$status = $redis->rename( 'price.mobile.Nokia.N8', 'price.mobile.Nokia.N8Duos' );
print $status."\n"; 
```
