# Функции defined и undef в perl — как с ними работать

## Функция defined

```perl
defined EXPR
defined
```

Функция возвращает булево значение, и отвечает на вопрос: содержит EXPR определенное значение или нет.

Функция вернет <font color="#00aa00">false</font>, если переменная еще не была инициализирована или ей было присвоено значение <font color="#00aa00">undef</font>. <font color="#00aa00">True</font> - если переменная была инициализирована любым значением, в том числе ей может быть присвоено значение числового нуля, пустой строки или строки содержащей символ нуля.

Именно этим <font color="#00aa00">defined</font> отличается от стандартной булевой проверки, которая не может отличить <font color="#00aa00">undef</font> от <font color="#00aa00">"0"</font>.

```perl
my $variable = "0";

if (defined $variable) {
	print "defined";
} else {
	print "undef";
}

if ($variable) {
	print "defined";
} else {
	print "undef";
}
```

Результат:
<pre>
% perl defi.pl
defined
undef
</pre>

## Функция undef

```perl
undef EXPR
undef
```

Функция <font color="#00aa00">undef</font> всегда возвращает неопределенное значение.  Если передать функции в качестве аргумента переменную - она присвоит ей неопределенное значение.

```perl
undef $variable;
# или так
$variable = undef;
```

Есть небольшой интересный нюанс в использовании <font color="#00aa00">undef</font> - не стоит использовать эту функцию для сравнения с переменными. Сравнение заданной переменной будет выполняться с числовым нулем или пустой строкой, а не с неопределенным значением. Результатом могут стать парадоксальные ситуации и ошибки.

Пример:

```perl
my $variable = 0;

if ($variable == undef) {
	print "var is undef";
} else {
	print "var is defined";
}

if (defined $variable) {
	print "var is defined";
} else {
	print "var is undef";
}
```

Вывод:
<pre>
% perl defi.pl
var is undef
var is defined
</pre>
