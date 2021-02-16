# Работа со стандартными потоками ввода-вывода в Unix - 2. Каналы

*Конспект, заметки по работе со стандартными потоками ввода-вывода, работа с каналами. Примеры проверены на Debian Linux. Буферизация STDOUT и STDERR в perl. Использование /dev/null . Mknod и mkfifo.*

## Неименованые каналы

Стандартные потоки можно перенаправлять не только в файлы, но и на вход других программ. Если поток вывода одной программы соединить с потоком ввода другой программы, получится конструкция, называемая каналом, конвейером.

В bash канал выглядит как последовательность команд, отделенных друг от друга символом **|** :
<font color="#1f6313"><pre>command1 | command2 | command3 ... </pre></font>
Стандартный поток вывода command1 подключается к стандартному потоку ввода command2, стандартный поток вывода command2 в свою очередь подключается к потоку ввода command3 и т.д.

Программы, образующие канал, выполняются параллельно как независимые процессы.

**Пример 1**

Создадим perl-скрипт, который будет принимать данные на вход.
<font color="#1f6313"><pre>while (<stdin>) {
  chomp;
  print length($_)."\n";
}
</stdin></pre></font>
Это простая программа, которая считывает строки со стандартного входа и печатает для каждой строки количество символов. Если запустить программу как есть, то она будет ожидать ввода строки с терминала, печатать количество введенных символов, и считывать следующую строку, до тех пор, пока вам это не надоест и вы не введете Ctrl+C.
<font color="#1f6313"><pre>$ perl stdin.pl
565757
6
^C
</pre></font>
Либо, можно использовать канал и подать программе на вход какой-нибудь файл, чтобы она подсчитала количество символов в его строках.
<font color="#1f6313"><pre>$ cat file.txt | perl stdin.pl
</pre></font>
<font color="#1f6313"><pre>$ ps -u root -f
root     20120 19879  0 09:18 pts/1    00:00:00 perl stdin.pl

$ lsof -a -p 20120
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF       NODE NAME
perl    20120 root  cwd    DIR 144,52     4096    2752575 /root/perl
perl    20120 root  rtd    DIR 144,52     4096    2528296 /
perl    20120 root  txt    REG 144,52  1487332    4489619 /usr/bin/perl
...
perl    20120 root    0r  FIFO    0,8      0t0 2764543421 pipe
perl    20120 root    1u   CHR  136,1      0t0          4 /dev/pts/1
perl    20120 root    2u   CHR  136,1      0t0          4 /dev/pts/1
perl    20120 root    7w  FIFO    0,8      0t0 2762752611 pipe
</pre></font>
К сожалению, процесс **cat** выполняется очень быстро, передает данные в канал и завершается, поэтому его данные тут не указаны.

**Пример 2**

Создадим программу, которая передает данные другой программе. Для связи между ними будет использован неименованый канал.
<font color="#1f6313"><pre>sleep(60);
print STDOUT "Standart stdout\n";
print STDOUT "Standart stdout 2\n";
</pre></font>
Запускаем perl-скрипт и передаем сгенерированные данные утилите **wc**:
<font color="#1f6313"><pre>$ perl stdout.pl | wc
      2       5      34
</pre></font>
Утилита wc, в своем самом простом варианте применения, подсчитывает количество строк, слов и байт в заданном файле, или в том тексте, который был передан ей на вход.
<font color="#1f6313"><pre>$ ps -u root -f
root     20008 19879  0 08:55 pts/1    00:00:00 perl stdout.pl
root     20009 19879  0 08:55 pts/1    00:00:00 wc

$ lsof -a -p 20008
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF       NODE NAME
perl    20008 root  cwd    DIR 144,52     4096    2752575 /root/perl
perl    20008 root  rtd    DIR 144,52     4096    2528296 /
perl    20008 root  txt    REG 144,52  1487332    4489619 /usr/bin/perl
...
perl    20008 root    0u   CHR  136,1      0t0          4 /dev/pts/1
perl    20008 root    1w  FIFO    0,8      0t0 2763569501 pipe
perl    20008 root    2u   CHR  136,1      0t0          4 /dev/pts/1
perl    20008 root    7w  FIFO    0,8      0t0 2762752611 pipe

$ lsof -a -p 20009
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF       NODE NAME
wc      20009 root  cwd    DIR 144,52     4096    2752575 /root/perl
wc      20009 root  rtd    DIR 144,52     4096    2528296 /
wc      20009 root  txt    REG 144,52    38524    4489933 /usr/bin/wc
...
wc      20009 root    0r  FIFO    0,8      0t0 2763569501 pipe
wc      20009 root    1u   CHR  136,1      0t0          4 /dev/pts/1
wc      20009 root    2u   CHR  136,1      0t0          4 /dev/pts/1
wc      20009 root    7w  FIFO    0,8      0t0 2762752611 pipe
</pre></font>
Если обратить внимание, то можно заметить, что система создала оба процесса одновременно, у них один и тот же родитель.Кроме того, был создан неименованый канал. Этот канал скрипт использует вместо */dev/pts/1 (файл, псевдоустройство - псевдотерминал, PTS, pseudo-terminal slave)*, в качестве стандартного вывода, и тот же самый канал указан для **wc** вместо */dev/pts/1*, в качестве стандартного входа.

**Пример 3**

Из perl-скрипта вызывается на выполнение **netstat**, и из этого же скрипта создается канал для связи с ней. Символ вертикальной черты в функции **open**, как раз указывает на то, что будет создан канал.
<font color="#1f6313"><pre>open(PIPE, "netstat -an 2&gt;/dev/null |") or die "can not fork: $!";

while(<pipe>) {
  print $_ if $_ =~ /^(tcp|udp|unix)/;
}

close(PIPE);
</pipe></pre></font>
Во время выполнения скрипта, можно заметить, что в дополнение к стандартным потокам ввода-вывода, для данного скрипта был добавлен четвертый - канал.
<font color="#1f6313"><pre>$ ps -u root -f
root     20705 19879  0 09:48 pts/1    00:00:00 perl pipe.pl
root     20706 20705  0 09:48 pts/1    00:00:00 [sh] &lt;defunct&gt;

$ lsof -a -p 20705
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF       NODE NAME
perl    20705 root  cwd    DIR 144,52     4096    2752575 /root/perl
perl    20705 root  rtd    DIR 144,52     4096    2528296 /
perl    20705 root  txt    REG 144,52  1487332    4489619 /usr/bin/perl
...
perl    20705 root    0u   CHR  136,1      0t0          4 /dev/pts/1
perl    20705 root    1u   CHR  136,1      0t0          4 /dev/pts/1
perl    20705 root    2u   CHR  136,1      0t0          4 /dev/pts/1
perl    20705 root    3r  FIFO    0,8      0t0 2766639351 pipe
perl    20705 root    7w  FIFO    0,8      0t0 2762752611 pipe
</defunct></pre></font>

## Именованные каналы

Неименованый канал можно построить только между процессами, которые порождены от одного процесса (и на практике они должны быть порождены одновременно, а не последовательно, хотя теоретически это не обязательно). Если же процессы имеют разных родителей, то между ними обычный, безымянный канал построить не получится.

Для решения этой задачи используются именованные каналы **fifo** (first in, first out). Они во всём повторяют обычные каналы (pipe), только имеют привязку к файловой системе. Создать именованный канал можно командой **mkfifo**:
<font color="#1f6313"><pre>$ mkfifo /tmp/fifo</pre></font>

Созданный канал можно использовать для соединения процессов между собой.
<font color="#1f6313"><pre>$ programm1 &gt; /tmp/fifo &amp; programm2 &lt; /tmp/fifo</pre></font>


**Пример использования именованого канала**

Для теста будем использовать уже известный скрипт *stdin.pl*:
<font color="#1f6313"><pre>while (<stdin>) {
  chomp;
  print length($_)."\n";
}
</stdin></pre></font>
Кроме того, создаем текстовый файл *file.txt*, который содержит пару строк:
<font color="#1f6313"><pre>hello,
my darling!
</pre></font>
Создаем именованый канал:
<font color="#1f6313"><pre>$ mkfifo mypipe

$ ls -la
drwxr-xr-x  2 root root 4096 Мар 26 09:24 .
drwx------ 12 root root 4096 Мар 26 02:40 ..
-rw-r--r--  1 root root   18 Мар 26 09:11 file.txt
prw-r--r--  1 root root    0 Мар 26 09:26 mypipe
-rw-r--r--  1 root root   64 Мар 26 09:18 stdin.pl
</pre></font>
Далее, вызываем утилиту **cat** для того, чтобы она прочитала данные из файла *file.txt* и передала их на вход каналу *mypipe*, и одновременно вызываем perl-скрипт *stdin.pl*, которому поручаем забрать данные из канала и обработать их.
<font color="#1f6313"><pre>$ cat file.txt &gt; mypipe &amp; perl stdin.pl &lt; mypipe
[1] 20192
6
11
[1]+  Done          cat file.txt &gt; mypipe
</pre></font>
Пока скрипт отрабатывается, мы имеем возможность посмотреть, с какими файловыми дескрипторами работает *stdin.pl*, и убедиться, что канал *mypipe* действительно передает данные на вход скрипту:
<font color="#1f6313"><pre>$ ps -u root -f
root     20229 19879  0 09:30 pts/1    00:00:00 perl stdin.pl

$ lsof -a -p 20229
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF       NODE NAME
perl    20229 root  cwd    DIR 144,52     4096    2752575 /root/perl
perl    20229 root  rtd    DIR 144,52     4096    2528296 /
perl    20229 root  txt    REG 144,52  1487332    4489619 /usr/bin/perl
...
perl    20229 root    0r  FIFO 144,52      0t0    2752585 /root/perl/mypipe
perl    20229 root    1u   CHR  136,1      0t0          4 /dev/pts/1
perl    20229 root    2u   CHR  136,1      0t0          4 /dev/pts/1
perl    20229 root    7w  FIFO    0,8      0t0 2762752611 pipe
</pre></font>

