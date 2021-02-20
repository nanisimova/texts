# Devel::InnerPackage

Модуль предоставляет всего одну функцию <font color="#00aa00">list_packages()</font>, которая ищет все вложенные пакеты в указанном пакете и возвращает их названия в виде списка.

**Синтаксис**

```perl
use Foo::Bar;
use Devel::InnerPackage qw(list_packages);

my @inner_packages = list_packages('Foo::Bar');
```

**Пример**

Допустим, есть пакет <font color="#00aa00">MyPackage</font>, который содержит внутри еще несколько пакетов:

```perl
package MyPackage;
sub func {}

package MyPackage::MyBasket;
sub func1 {}

package MyPackage::MyService;
sub func2 {}

1;
```

В отдельном скрипте выполняем вызов функции <font color="#00aa00">list_packages()</font>:

```perl
use MyPackage;
use Devel::InnerPackage qw(list_packages);

my @list = list_packages("MyPackage");

print join(', ', @list);
```

Результат выполнения:
<pre>
$ perl Devel_InnerPackage.pl
MyPackage::MyService, MyPackage::MyBasket
</pre>
Поиск входящих пакетов осуществляется только в указанном пакете, в файле этого пакета. В других файлах поиск не выполняется. Функция выполняет поиск с помощью <font color="#00aa00">grep()</font>, <font color="#00aa00">substr()</font> и регулярных выражений.

