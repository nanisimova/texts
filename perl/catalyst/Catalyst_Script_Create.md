# Catalyst::Script::Create

Catalyst::Script::Create служит для создания новых компонент Catalyst : моделей, представлений или контроллеров. Если какие-то файлы уже существут, они не будут перезаписаны. Новые будут созданы с окончанием ".new". Если требуется перезаписать существующие файлы, можно использовать <font color="#00aa00">--force</font>.

<pre>myapp_create.pl [options] model|view|controller name [helper] [options]

 Options:
   --force        don't create a .new file where a file to be created exists
   --mechanize    use Test::WWW::Mechanize::Catalyst for tests if available
   --help         display this help and exits

 Examples:
   myapp_create.pl controller My::Controller
   myapp_create.pl controller My::Controller BindLex
   myapp_create.pl --mechanize controller My::Controller
   myapp_create.pl view My::View
   myapp_create.pl view MyView TT
   myapp_create.pl view TT TT
   myapp_create.pl model My::Model
</pre>

Пример:
<pre>perl script/myapp_create.pl view TT TT</pre>

Когда <font color="#00aa00">myapp_create.pl</font> получает задачу создать новую компоненту, он вызывает для этого Catalyst::ScriptRunner, Catalyst::ScriptRunner вызывает Catalyst::Script::Create, Catalyst::Script::Create вызывает Catalyst::Helper, который выполнив определенную работу, вызывает модуль-помощник для создания конкретной компоненты.

Модуль Catalyst::Script::Create содержит только один метод <font color="#00aa00">run()</font>, который создает объект Catalyst::Helper и вызывает для него метод <font color="#00aa00">mk_component()</font>, передавая ему аргументы командной строки.
