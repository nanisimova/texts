# Как изменить окончания строк и удалить в тексте ^M . Как изменить кодировку файла

Файлы в ASCII-кодировке или совместимом наборе символов, для обозначения конца строки используют символы:
<ul>
 	<li>LF (от англ. Line feed (перевод строки), 0x0A)</li>
 	<li>CR (от англ. Carriage Return, 0x0D)</li>
 	<li>CRLF (т.е. оба символа).</li>
</ul>
В windows-системах обычно используется CRLF, в unix - LF . В web-разработке преимущественно используются unix-системы, и как следствие, принято, чтобы в файлах окончания строк обозначались с помощью LF.

Тем не менее, иногда открываешь файл для редактирования и видишь ЭТО:
<pre>FOREACH row = services;^M
    SET service_row = {^M
        attr =&gt; {},^M
        list =&gt; [],^M
    };^M
^M
</pre>
Это означает, что кто-то отредактировал файл в текстовом редакторе под Windows. Подобный формат файла может привести к проблемам. Кроме того, такие файлы очень неудобно редактировать, находясь в unix-среде. Случайное удаление ^M приведет к нарушению формата файла и невозможности его корректного чтения.


## Утилиты dos2unix и unix2dos

Для корректного преобразования формата файла удобно использовать утилиту dos2unix . При необходимости можно выполнить и обратное преобразование, с помощью unix2dos .

Установка:
<pre>$ sudo apt-get install dos2unix
$ sudo apt-get install unix2dos
</pre>
По умолчанию, утилита dos2unix перезапишет указанный файл. Но можно использовать ключ -n и тогда преобразованный текст будет сохранен в другом файле, название которого нужно указать в качестве второго аргумента.
<pre>$ dos2unix filename.txt
dos2unix: converting file filename.txt to Unix format ...
</pre>
Для массового преобразования файлов можно использовать маску:
<pre>$ dos2unix -k *.inc
</pre>
Ключ -k позволит оставить без изменений время создания файла.

Можно вместо конвертации исходного файла создать новый файл, который будет содержать данные в нужном формате:
<pre>$ dos2unix -n in.txt out.txt
</pre>
Использование unix2dos аналогично утилите dos2unix:
<pre>$ unix2dos filename.txt
</pre>

## Утилиты todos и fromdos

Данные утилиты, так же как утилиты dos2unix и unix2dos, выполняют конвертацию файлов из unix-формата в windows-формат, и обратно.

<i>Мне для работы удобно использовать dos2unix, поэтому fromdos я не устанавливала, но следует иметь ввиду такую возможность.</i>

## Утилита tr

<pre>$ cat service.inc | tr -d '\r' &gt; memo.txt</pre>
Т.е. утилита tr удалит символы "\r" из строк.

## Преобразование конца строк в файлах с помощью perl

Чтобы изменить windows-формат окончания строк  (CRLF) на unix-формат(LF), можно использовать perl:
<pre># perl -pi -e 's/\r\n/\n/;' filename.txt</pre>
Обратное преобразование из unix-формата (LF) в windows-формат (CRLF):
<pre># perl -pi -e 's/\n/\r\n/;' filename.txt</pre>
Для преобразования сразу нескольких файлов, можно использовать маску, вместо прямого указания имени файла:
<pre># perl -pi -e 's/\n/\r\n/;' b*</pre>
При указании маски, кавычки не используются, попытка их применения приведет к ошибке
<b>"No such file or directory"</b>:
<pre># perl -pi -e 's/\n/\r\n/;' "b*"
Can't open b*: No such file or directory.
</pre>
Если вы попробуете выполнить преобразование конца строк под windows, то получите ошибку
<b>"Can't do inplace edit without backup"</b>:
<pre>&gt; perl -pi -e 's/\r\n/\n/;' filename.txt
Can't do inplace edit without backup.
</pre>
Это связано с ограничениями windows. Нужно использовать вот такую команду:
<pre>perl -i.bak -p -e 's/\r\n/\n/;' filename.txt &amp;&amp; del filename.txt.bak</pre>


## Изменение кодировки файла с помощью утилиты iconv

Чтобы конвертировать файл in.txt из WINDOWS-1251 в UTF-8 out.txt, можно использовать утилиту iconv ( на примере Linux Ubunta ):
<pre># iconv -f WINDOWS-1251 -t UTF-8 in.txt &amp;&gt; out.txt</pre>
Чтобы посмотреть полный список кодировок, с которыми утилита может работать, нужно вызвать iconv с параметром --list :
<pre># iconv --list
The following list contains all the coded character sets known.  This does
not necessarily mean that all combinations of these names can be used for
the FROM and TO command line parameters.  One coded character set can be
listed with several different names (aliases).

  437, 500, 500V1, 850, 851, 852, 855, 856, 857, 860, 861, 862, 863, 864, 865,
  866, 866NAV, 869, 874, 904, 1026, 1046, 1047, 8859_1, 8859_2, 8859_3, 8859_4,
  8859_5, 8859_6, 8859_7, 8859_8, 8859_9, 10646-1:1993, 10646-1:1993/UCS4,
  ANSI_X3.4-1968, ANSI_X3.4-1986, ANSI_X3.4, ANSI_X3.110-1983, ANSI_X3.110,
...
</pre>

