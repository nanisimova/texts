# Что такое функция require в Perl?

Функция <font color="#00aa00">require</font> может вызываться с разными типами аргументов.

```perl
require VERSION
require EXPR
require
```

```perl
require 'pack.pl';
```

Если аргумент - это простая строка, <font color="#00aa00">require</font> считает, что ей передано имя файла. <font color="#00aa00">Require</font> попытается открыть файл, загрузить и выполнить содержащийся в нем код Perl. Если указанный файл уже был загружен, функция создаст исключительную ситуацию и предотвратит возникновение ошибки.

Подключаемый с помощью <font color="#00aa00">require</font> файл должен иметь в конце символ <font color="#00aa00">"1;"</font>.

```perl
require 5.004;
```

Если аргумент, переданный <font color="#00aa00">require</font> - это номер версии, <font color="#00aa00">require</font> потребует, чтобы номер версии perl для запуска кода был не ниже указанного.
```perl
require Pack;
```

Если в качестве аргумента <font color="#00aa00">require</font> передано голое имя пакета, функция будет предполагать, что у искомого файла должно быть расширение <font color="#00aa00">.pm</font> и обрабатывать разделители пакетов <font color="#00aa00">"::"</font> как разделители каталогов.
<p style="margin-top: 20px;">Свою работу <font color="#00aa00">require</font> выполняет на этапе исполнения программы. В отличие от <font color="#00aa00">use</font>, которая активна на этапе компиляции.</p>
<font color="#00aa00">Require не выполняет импорта подпрограмм из пакетов!</font> Если вы попытаетесь выполнить подпрограмму из пакета, подключенного функцией <font color="#00aa00">require</font>, вас ожидает неудача и ошибка:

<pre>
% perl require.pl
Undefined subroutine &amp;main::print_text called at require.pl line 4.
</pre>
