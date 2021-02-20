# DBIx::Class — Как создать схему БД

*Когда я впервые познакомилась с DBIx, впечатления были самые матерные. Но ничего, привыкла. Начала даже находить в ORM логику и смысл.*

## Создание схемы для DBIx::Class

Создание схемы для DBIx состоит из нескольких этапов:

<ol>
<li>Нужно создать Базу Данных и таблицы в ней</li>
<li>Установить DBIx::Class</li>
<li>Установить модуль DBIx::Class::Schema::Loader</li>
<li>Запустить dbicdumpНа unix-системе команда будет выглядеть примерно так:
<pre>dbicdump -o dump_directory=./lib My::App::Schema 
   'dbi:mysql:mydb:localhost:3306' user password
</pre>
И будет вам счастье.

А вот если вы, как я, решите создать схему DBIx под windows... Windows работать с такой командой отказалась категорически. Ситуацию спасло создание конфига, который можно скормить dbicdump.

Содержимое конфига 1.conf :

```
schema_class MyApp::Schema
#lib /extra/perl/libs

# connection string
<connect_info>
    dsn     dbi:mysql:test
    user    root
    pass    
</connect_info>
```

Запускаем dbicdump:

<pre>
C:\strawberry\perl\site\bin&gt;dbicdump 1.conf
Dumping manual schema for MyApp::Schema to directory . ...
Schema dump completed.
</pre>
</li>
</ol>

