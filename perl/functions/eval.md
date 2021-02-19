# Что такое функция eval в perl и как ее использовать

<pre>
eval EXPR
eval BLOCK
</pre>
При работе с функцией <font color="#00aa00">eval</font>, можно использовать 2 варианта записи функции: <font color="#00aa00">eval BLOCK</font> и <font color="#00aa00">eval EXPR</font>. Каждый из вариантов имеет свои особенности в использовании.

## eval BLOCK

Если задан <font color="#00aa00">eval BLOCK</font>, функция выполняет перехват исключительных ситуаций, которые привели бы к возникновению фатальной ошибки и аварийному завершению программы.

<font color="#00aa00">eval BLOCK</font> является альтернативой оператору "try" в Python, Java, и т.п.

Функция <font color="#00aa00">eval</font> возвращает значение, полученное в результате последнего проведенного вычисления. Если во время выполнения <font color="#00aa00">eval</font> возникает исключительная ситуация, то <font color="#00aa00">eval</font> возвращает undef, и помещает в переменную <font color="#00aa00">$@</font> текст произошедшей ошибки. Если ошибка не возникла, значение переменной <font color="#00aa00">$@</font> будет равно пустой строке. Для возврата значений можно использовать в теле <font color="#00aa00">BLOCK</font> функцию <font color="#00aa00">return</font>.

Код, размещенный в <font color="#00aa00">BLOCK</font>, проверяется и компилируется одновременно с компиляцией всей программы.

Стандартный пример использования <font color="#00aa00">eval BLOCK</font>:
<pre>eval { #... };

if ($@) { #... };
</pre>
Форма <font color="#00aa00">eval BLOCK</font> работает значительно быстрее, чем <font color="#00aa00">eval EXPR</font>.

При равных условиях, использование формы <font color="#00aa00">eval BLOCK</font> предпочтительнее, чем <font color="#00aa00">eval EXPR</font>.

### Пример использования eval BLOCK

С помощью <font color="#00aa00">eval BLOCK</font> очень удобно проверять доступность тех или иных модулей:

```perl
eval {
        require PDF::API3;
};
if ($@) {
        print "Need install module PDF::API3";
}
```

Можно использовать <font color="#00aa00">eval BLOCK</font>, если есть сомнения в возможности вызова подпрограмм:

```perl
use My::Module;

my $my_object = eval {My::Module->new};
if ($@) {
        print "Error";
        exit;
}
$my_object->my_method;
```

или так:

```perl
eval { 
	$obj->create_db_connect($user, $pass);
	$obj->run_mode("update");
};
```

## eval EXPR

<font color="#00aa00">eval EXPR</font> компилирует и выполняет код заданного выражения на этапе исполнения программы. Если во время выполнения <font color="#00aa00">eval EXPR</font> возникает исключительная ситуация, функция возвращает undef, в переменную <font color="#00aa00">$@</font> передается текст произошедшей ошибки.

Если исключительная ситуация не возникла, <font color="#00aa00">eval</font> возвращает результат последнего вычисления.

Использовать <font color="#00aa00">eval</font> надо осторожно. Если функции <font color="#00aa00">eval</font> передаются введенные пользователем данные - эти данные нужно тщательно проверять, во избежание запуска вредоносного кода и взлома системы.

### Пример использования eval EXPR

<font color="#00aa00">eval EXPR</font> можно использовать, если заранее не известно, какие команды и подпрограммы потребуется выполнить.

```perl
eval "require $ARGV[0]";

if ($@) {
        print "Need install $ARGV[0] \n\n";
}
```

