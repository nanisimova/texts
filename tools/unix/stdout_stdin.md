# Работа со стандартными потоками ввода-вывода в Unix - 1

*Конспект, заметки по работе со стандартными потоками ввода-вывода. Примеры проверены на Debian Linux. Буферизация STDOUT и STDERR в perl. Использование /dev/null . Mknod и mkfifo.*


## Стандартные потоки ввода-вывода: STDIN, STDOUT, STDERR

Процесс взаимодействия программы с окружением выполняется в терминах записи и чтения в файл. Так, вывод на экран представляется как запись в файл, а ввод данных — как чтение файла. Файл, из которого осуществляется чтение, называется стандартным потоком ввода, а в который осуществляется запись — стандартным потоком вывода.

Кроме потоков ввода и вывода, существует еще и стандартный поток ошибок, на который выводятся все сообщения об ошибках и те информативные сообщения о ходе работы программы, которые не могут быть выведены в стандартный поток вывода.

Большинство программ, которые работают с потоками ввода и вывода, работают с ними как с простыми файлами, и не рассчитывают на то, что поток подключен к терминалу.

Вывод данных на экран и чтение их с клавиатуры происходит потому, что по умолчанию стандартные потоки ассоциированы с терминалом пользователя. Это не является обязательным — потоки можно подключать к чему угодно — к файлам, программам и даже устройствам - эта операция называется перенаправлением.

Стандартные потоки привязаны к файловым дескрипторам с номерами 0, 1 и 2.
<ul>
<li>Стандартный поток ввода (STDIN) — 0;</li>
<li>Стандартный поток вывода (STDOUT) — 1;</li>
<li>Стандартный поток ошибок (STDERR) — 2.</li>
</ul>

**Пример**

Создадим очень простую perl-программу, которая будет отправлять информацию в STDOUT и STDERR. И используем ее в дальнейшем, для проверки разных способов перенаправления вывода.
<font color="#1f6313"><pre>sleep(60);
print STDOUT "Standart stdout\n";
print STDERR "Some error!\n";
</pre></font>

Sleep используется для того, чтобы успеть получить информацию у запущенном процессе.

С помощью **lsof** просматриваем список открытых файлов и файловых дескрипторов для запущенной программы. Эта информация может оказаться полезной и интересной, когда дело доходит до работы с каналами и сокетами.

<font color="#1f6313"><pre># lsof -a -p 18656
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF       NODE NAME
perl    18656 root  cwd    DIR 144,52     4096    2752575 /root/perl
perl    18656 root  rtd    DIR 144,52     4096    2528296 /
perl    18656 root  txt    REG 144,52  1487332    4489619 /usr/bin/perl
perl    18656 root  mem    REG    8,3             4489619 /usr/bin/perl (path dev=144,52)
perl    18656 root  mem    REG    8,3             4457085 /usr/lib/locale/locale-archive (path dev=144,52)
perl    18656 root  mem    REG    8,3             6565272 /lib/i386-linux-gnu/libcrypt-2.13.so (path dev=144,52)
perl    18656 root  mem    REG    8,3             6565131 /lib/i386-linux-gnu/libc-2.13.so (path dev=144,52)
perl    18656 root  mem    REG    8,3             6565158 /lib/i386-linux-gnu/libpthread-2.13.so (path dev=144,52)
perl    18656 root  mem    REG    8,3             6565127 /lib/i386-linux-gnu/libm-2.13.so (path dev=144,52)
perl    18656 root  mem    REG    8,3             6565134 /lib/i386-linux-gnu/libdl-2.13.so (path dev=144,52)
perl    18656 root  mem    REG    8,3             6565284 /lib/i386-linux-gnu/ld-2.13.so (path dev=144,52)
perl    18656 root    0u   CHR  136,1      0t0          4 /dev/pts/1
perl    18656 root    1u   CHR  136,1      0t0          4 /dev/pts/1
perl    18656 root    2u   CHR  136,1      0t0          4 /dev/pts/1
perl    18656 root    7w  FIFO    0,8      0t0 2738690077 pipe
</pre></font>
Кстати, следует учитывать тот факт, что по умолчанию, perl буферизирует вывод STDOUT, но не буферизирует STDERR.
Попытка выполнения вот такого кода:
<font color="#1f6313"><pre>print STDOUT "Standart stdout ";
print STDERR "Some error! ";
print STDOUT "Standart stdout 2 ";
</pre></font>
может породить вот такую ситуацию:
<font color="#1f6313"><pre>perl stdout.pl
Some error! Standart stdout Standart stdout 2
</pre></font>
Текст об ошибке оказался в начале строки, несмотря на то, что должен был оказаться в середине.

То же самое произойдет, если попытаться выполнить код:
<font color="#1f6313"><pre>print time();
sleep(20);
</pre></font>
В данном случае, сначала программа подождет 20 секунд, и только потом на экране появится сообщение. Хотя на самом деле, сообщение будет отправлено в STDOUT раньше, чем будет вызван sleep. Когда встречаешь подобное поведение системы в первый раз, оно может озадачить.

Отключить буферизацию можно, используя переменную **$|** и присвоив ей значение **1**.
<font color="#1f6313"><pre>$| = 1;
</pre></font>
Другой вариант - в конце каждой строки добавлять "\n". Если записать код программы следующим образом:
<font color="#1f6313"><pre>print STDOUT "Standart stdout\n";
print STDERR "Some error!\n";
print STDOUT "Standart stdout 2\n";
</pre></font>
результат выполнения будет именно таким, как ожидается:
<font color="#1f6313"><pre># perl stdout.pl
Standart stdout
Some error!
Standart stdout 2
</pre></font>
&nbsp;


## Перенаправление потоков ввода-вывода

**&lt; filename** -> Используем указанный файл как источник данных для стандартного потока ввода.

**&gt; filename** -> Перенаправляем стандартный поток вывода в указанный файл. Если файл не существует - он будет создан, если существует - перезаписан.

<font color="#1f6313"><pre>$ perl stdout.pl &gt; file.txt
Some error!
$ less file.txt
Standart stdout
</pre></font>
В данном случае, в файл перенаправлены только те данные, которые подавались на STDOUT. Поток STDERR по прежнему связан с терминалом и выдает данные на него.

Чтобы обнулить содержимое файла, можно использовать команду:
<font color="#1f6313"><pre>$ &gt; file.txt
</pre></font>

**2&gt; filename** - Перенаправляем стандартный поток ошибок в указанный файл. Если файл не существует - он будет создан, если существует - перезаписан.

<font color="#1f6313"><pre>$ perl stdout.pl 2&gt; file.txt
Standart stdout
$ less file.txt
Some error!
</pre></font>
При этом, поток STDOUT продолжает выводить данные на терминал.

В данном случае, "2" - это идентификатор потока ввода-вывода. Идентификатор с номером 2, обычно соответствует потоку STDERR, но можно указать номер любого другого потока вывода.

Вызов *"perl stdout.pl 1&gt; filename"* эквивалентен вызову *"perl stdout.pl &gt; filename"*.

Можно оба потока вывода перенаправить в файлы:
<font color="#1f6313"><pre>$ perl stdout.pl &gt;fileout 2&gt; fileerr
& ls
fileerr  fileout  stdout.pl
</pre></font>
Если выводимые данные не требуются и вы не хотите их просматривать в дальнейшем, можно перенаправить вывод в /dev/null .
<font color="#1f6313"><pre>$ perl stdout.pl &gt; /dev/null
Some error!
</pre></font>


**&gt;&gt;filename** - Перенаправляем стандартный поток вывода в заданный файл. Если файл не существует - он будет создан, если существует - данные будут дописаны в конец файла.
<font color="#1f6313"><pre>$ perl stdout.pl &gt; log
Some error!
$ perl stdout.pl &gt;&gt; log
Some error!
$ tail log
Standart stdout
Standart stdout 2
Standart stdout
Standart stdout 2
</pre></font>

**2&gt;&gt;filename** - Перенаправляем стандартный поток ошибок в указанный файл. Если файл не существует - он будет создан. Если существует - данные будут дописаны в конец файла.

**&amp;&gt;filename, &gt;&amp;filename** - Перенаправляем и стандартный поток вывода, и стандартный поток ошибок в указанный файл.
<font color="#1f6313"><pre>$ perl stdout.pl &gt;&amp; log
$ tail log
Some error!
Standart stdout
Standart stdout 2
</pre></font>

**&gt;&amp;-** - Закрываем стандартный поток вывода.
<font color="#1f6313"><pre>$ perl ipc.pl &gt;&amp;-</pre></font>
<font color="#1f6313"><pre>$ ps -u root -f
root     19903 19879  0 08:38 pts/1    00:00:00 perl ipc.pl

$ lsof -a -p 19903
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF       NODE NAME
perl    19903 root  cwd    DIR 144,52     4096    2752575 /root/perl
perl    19903 root  rtd    DIR 144,52     4096    2528296 /
perl    19903 root  txt    REG 144,52  1487332    4489619 /usr/bin/perl
...
perl    19903 root    0u   CHR  136,1      0t0          4 /dev/pts/1
perl    19903 root    1r   REG 144,52       38    2753000 /root/perl/ipc.pl
perl    19903 root    2u   CHR  136,1      0t0          4 /dev/pts/1
perl    19903 root    7w  FIFO    0,8      0t0 2762752611 pipe
</pre></font>
В данном случае, для скрипта был отключен STDOUT. Скрипт был выполнен, но никаких надписей на экране терминала при этом сделано не было.

**2&gt;&amp;-** - Закрываем стандартный поток вывода ошибок.
<font color="#1f6313"><pre># perl ipc.pl 2&gt;&amp;-
1427373871
</pre></font>
В данном случае, закрыт был поток вывода ошибок, но стандартный вывод отработал как обычно.
<font color="#1f6313"><pre>$ ps -u root -f
root     19938 19879  0 08:44 pts/1    00:00:00 perl ipc.pl
$ lsof -a -p 19938
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF       NODE NAME
perl    19938 root  cwd    DIR 144,52     4096    2752575 /root/perl
perl    19938 root  rtd    DIR 144,52     4096    2528296 /
perl    19938 root  txt    REG 144,52  1487332    4489619 /usr/bin/perl
...
perl    19938 root    0u   CHR  136,1      0t0          4 /dev/pts/1
perl    19938 root    1u   CHR  136,1      0t0          4 /dev/pts/1
perl    19938 root    2r   REG 144,52       38    2753000 /root/perl/ipc.pl
perl    19938 root    7w  FIFO    0,8      0t0 2762752611 pipe
</pre></font>


## /dev/null
**/dev/null** - это символьное псевдоустройство. Можно обращаться к нему, как к файлу. Не обязательно использовать **/dev/null** , можно создать свой собственный null-файл с помощью утилиты mknod. Mknod - предназначена для создания специальных символьных или блочных файлов. Кроме того, mknod можно использовать для создания именованых каналов.

<font color="#1f6313"><pre>mknod [OPTION]... NAME TYPE [MAJOR MINOR]</pre></font>
*mknod FILE c 1 3* - создает null-файл.

<font color="#1f6313"><pre>$ mknod NULLFILE c 1 3
$ ls -la
drwxr-xr-x  2 root root 4096 Мар 26 05:51 .
drwx------ 12 root root 4096 Мар 26 02:40 ..
crw-r--r--  1 root root 1, 3 Мар 26 05:51 NULLFILE
-rw-r--r--  1 root root   75 Мар 26 02:56 stdout.pl
</pre></font>

<font color="#1f6313"><pre>$ perl stdout.pl 2&gt; NULLFILE
Standart stdout
</pre></font>

**/dev/null** очень полезен, когда требуется запустить скрипт, выдающий множество предупреждений в STDERR, особенно если этот скрипт запущен через cron, а cron, в свою очередь, шлет огромные текстовые сообщения вам на почту:
<font color="#1f6313"><pre>$ crontab -l
00 10 * * * perl $HOME/trunk/bin/table_update.pl 2&gt;/dev/null
</pre></font>
С помощью **/dev/null** можно обнулить содержимое файла:
<font color="#1f6313"><pre>$ cp /dev/null newfile
</pre></font>




