# File::Basename — парсим file paths

Модуль <font color="#00aa00">File::Basename</font> - используется для распарсивания пути к файлу, имени файла, расширения файла.

Модуль предоставляет для работы несколько функций, далее будут рассмотрены три из них: <font color="#00aa00">fileparse()</font>, <font color="#00aa00">basename()</font> и <font color="#00aa00">dirname()</font> .

Функции <font color="#00aa00">dirname()</font> и <font color="#00aa00">basename()</font> имитируют работу одноименных функций в <font color="#00aa00">C</font> и <font color="#00aa00">shell</font>. Поэтому, имеет смысл, просмотреть документацию к каждой из функций.

## Функция basename()

Функция <font color="#00aa00">basename()</font> возвращает последний элемент заданного пути, не важно, что это будет - имя файла или каталога.

Можно указать суффикс, который будет исключен из возвращаемого результата. Данная особенность очень удобна, когда требуется получить только имя файла, без расширения.

Пример работы с функцией <font color="#00aa00">basename()</font>:

```perl
use File::Basename;

my $filename = basename('/home/www/dev-lab/cgi');
print $filename; # выведет: 'cgi'

$filename = basename('/home/www/dev-lab/cgi/test_name.pl');
print $filename; # выведет: 'test_name.pl'
```

Пример работы с расширениями файлов:

```perl
use File::Basename;

my $filename = basename('/home/www/dev-lab/cgi/test_name.pl', '.pl');
print $filename; # выведет: 'test_name'

$filename = basename('/home/www/dev-lab/cgi/test_name.pl', '.txt', '.pl');
print $filename; # выведет: 'test_name'

my @array = (".pl", ".cgi", ".txt");
$filename = basename('/home/www/dev-lab/cgi/test_name.pl', @array);
print $filename; # выведет: 'test_name'
```

## Функция dirname()

В противоположность <font color="#00aa00">basename()</font>, функция <font color="#00aa00">dirname()</font> возвращает не последний элемент полученного пути, а все элементы, кроме последнего. В зависимости от используемой операционной системы, результаты работы функции могут различаться.

```perl
use File::Basename;

my $filename = dirname('/home/www/dev-lab/cgi/test_name.pl');
print $filename; # выведет: '/home/www/dev-lab/cgi'

$filename = dirname('/home/www/dev-lab/cgi/');
print $filename; # выведет: '/home/www/dev-lab'

$filename = dirname('/home/www/dev-lab/cgi');
print $filename; # выведет: '/home/www/dev-lab'

$filename = dirname('bin');
print $filename; # выведет: '.'

$filename = dirname('../bin');
print $filename; # выведет: '..'
```

## Функция fileparse()

Функция <font color="#00aa00">fileparse()</font> - делит любой заданный путь на три элемента - "имя файла", "каталог" и "суффикс", и возвращает их в виде массива. Массив всегда содержит три элемента, которые иногда могут иметь пустые значения.

Пример работы с функцией <font color="#00aa00">fileparse()</font>:

```perl
use File::Basename;

my @path = fileparse("/home/www/dev-lab/cgi");
print join(' ', @path)."\n";
# выведет: 'cgi /home/www/dev-lab/ ' 

my @path2 = fileparse("/home/www/dev-lab/cgi/test_name.pl");
print join(' ', @path2)."\n";
# выведет: 'test_name.pl /home/www/dev-lab/cgi/ ' 
```

```perl
my @path = fileparse("/home/www/dev-lab/cgi/");
print join(' | ', @path)."\n";
# выведет: ' | /home/www/dev-lab/cgi/ | '
```

Пример работы с расширениями файлов:

```perl
my @path = fileparse("/home/www/dev-lab/cgi/test_name.pl", ".pl");
print join(' ', @path)."\n";
# выведет: 'test_name /home/www/dev-lab/cgi/ .pl'
```
