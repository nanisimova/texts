# DBIx::Class. Создание записей в таблицах БД, их поиск, обновление и удаление. Примеры кода

## Поиск записей

Выполнение SELECT-запросов.

```perl
my $rs = $schema->resultset('Recipe')->search({
    author_id => $author_id,
  }, {
    order_by => { -desc => 'id' }, # ORDER BY id DESC
    rows => 10, page => 2, # LIMIT x, y
    columns => [qw/id name author_id/], # SELECT id, name, author_id
  });
```

Примечание: у postgresql работа с ограничениями на количество строк идет не только через LIMIT, но и OFFSET.

```perl
my $rs = $schema->resultset('User')->search({
    'me.id'  => '26753',
    'friend.id' => '345'
}, {
    join            => [ 'friends' ],
    prefetch        => [ 'companies', 'friends' ],
    custom_prefetch => [ 'hobbies' ],
   }
);
```

Для выполнения запроса SELECT * FROM table нужно выполнить запрос без параметров:

```perl
my $rs = $schema->resultset('User')->search({});
```

## Удаление записей

Простой DELETE-запрос:

```perl
$schema->resultset('Recipe')->find($id)->delete;
```

## Добавление записей

Простой INSERT-запрос:

```perl
my $user = $schema->resultset('User')->create({
    login => $login,
    email => $email || undef,
});
```

Если строка с заданными параметрами не найдена, выполняется INSERT-запрос. Если строка найдена - UPDATE. Поиск осуществляется по primary key или unique key.

```perl
$schema->resultset('User')->update_or_create({
    id    => $id,
    login => $login,
    email => $email || undef,
});
```

## Обновление записей

Простой UPDATE-запрос:

```perl
$schema->resultset('User')->search({ id => $form->{id} })->update({ delete_flag => 1 });
```

