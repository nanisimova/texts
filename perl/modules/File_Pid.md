﻿# Использование pid-файлов для предотвращения повторного запуска скрипта

*Использование модулей File::Pid, Pid::File::Flock и File::Flock::Tiny. Блокировки файлов, работа с pid-файлами. Работа только одной копии скрипта в один момент времени. Защита от повторного запуска одного и того же скрипта, до того, как первый экземпляр завершит свою работу.*

Допустим, что есть perl-скрипт, который забирает данные с удаленных серверов. Или разносит данные из одного источника в таблицы БД. Скрипт работает достаточно долго, от получаса и более - зависит от объема данных, скорости ответа удаленных серверов и т.п. Запускается по cron, раз в час.

Во избежание проблем с дублированием данных, взаимными блокировками в БД и т.п., необходимо чтобы скрипт всегда был запущен только в одном экземпляре.

Для этого можно использовать pid-файл. При запуске, скрипт проверяет наличие pid-файла. Если файл доступен, то блокирует его и сохраняет в файле свой PID, и только после этого приступает к выполнению своей основной работы. Если файл существует, но доступ к нему заблокирован - скрипт сразу же завершает свою работу. Если pid-файл не существует, то скрипт создает его, блокирует, сохраняет свой pid, и приступает к основной работе.

Для того, чтобы не изобретать велосипед, можно найти на cpan модули для работы с pid-файлами. В простых приложениях их будет вполне достаточно.

## File::Flock::Tiny

<font color="#00aa00">File::Flock::Tiny</font> - очень удобный модуль для работы с pid-файлами.

Установка проходит без проблем.
<pre># cpan install File::Flock::Tiny</pre>

Пример использования, pid2.pl:

```perl
#!/usr/bin/perl

use strict;
use File::Flock::Tiny;

my $pid = File::Flock::Tiny->write_pid('script.pid') or do {
  print "Script already running";
  exit(0);
};
sleep(60); # имитируем длительную работу скрипта

exit;
```

Сохраняем скрипт и запускаем его. После этого будет создан файл <font color="#00aa00">script.pid</font> с ID процесса внутри.
<pre>$ perl pid2.pl</pre>

Смотрим список процессов, убедиться, что скрипт работает:
<pre>$ ps -u root -f
root     31487 30877  0 02:32 pts/1    00:00:00 perl pid2.pl
</pre>

После этого, пытаемся в другом окне терминала запустить тот же самый скрипт, не дожидаясь окончания работы первого экземпляра:

<pre>$ perl pid2.pl
Script already running
</pre>

### Методы File::Flock::Tiny

**File::Flock::Tiny-&gt;lock($file)**

Эксклюзивная блокировка файла. <font color="#00aa00">$file</font> - может быть как именем файла, так и файловым дескриптором. Файл блокируется до того момента, пока процесс не выйдет из границ функции, в которой блокировка была установлена. Либо, можно вызвать метод разблокировки объекта. Если указанный файл не существовал - он будет создан.

Если скрипт был повторно запущен до того момента, как будет снята блокировка файла, второй процесс будет ожидать снятия блокировки и после этого продолжит работу, в свою очередь, захватив файл.

```perl
#!/usr/bin/perl

use strict;
use File::Flock::Tiny;

my $lock = File::Flock::Tiny->lock('script.txt');
$lock->write_pid;
exit;
```

**File::Flock::Tiny-&gt;trylock($file)**

Делает попытку заблокировать файл, если не получилось - возвращает <font color="#00aa00">undef</font>.

```perl
my $lock = File::Flock::Tiny->trylock('script.txt') or do {
  print "File already blocked: $";
  exit(0);
};
```

**File::Flock::Tiny-&gt;write_pid($file)**

Делает попытку заблокировать указанный файл и сохранить в него ID текущего процесса. Если попытка удалась - возвращает ссылку на заблокированный объект, если нет - вернет <font color="#00aa00">undef</font>. После завершения процесса файл не удаляется, но блокировка будет снята. При повторном выполнении файл будет вновь заблокирован, ID процесса в файле - обновлен.

**$lock-&gt;write_pid**

Удаляет все данные из заблокированного файла, если они были, и сохраняет в него ID текущего процесса.

**$lock-&gt;release**

Снимает блокировку и закрывает файл.

**$lock-&gt;close**

Метод закрывает файловый дескриптор, но блокировку с файла не снимает! Может быть использовано, если есть порожденные дочерние процессы, которым блокировка еще для чего-то будет нужна.

Однако, данный метод работает не на всех операционных системах, поэтому, во имя переносимости программ - его лучше не использовать.

## Pid::File::Flock

Пример использования, pid4.pl:

```perl
#!/usr/bin/perl

use strict;
use Pid::File::Flock;

Pid::File::Flock->new( name => "script3.pid" , dir => "." )->abandon;
exit;
```

Создает или открывает уже существующий pid-файл, устанавливает блокировку, сохраняет в него ID процесса.  После завершения работы блокировка снимается. Файл не удаляется.

При попытке запустить второй экземпляр скрипта, пока еще не завершил работу первый и с pid-файла не была снята блокировка, выдает ошибку:

```perl
# perl pid4.pl
found alive process (32403), exit at pid4.pl line 7.
```

### Методы Pid::File::Flock

**new( $path, %options )**

<font color="#00aa00">$path</font> - необязательный аргумент, который применяется, если параметры 'dir','name' and 'ext' по каким-либо причинам не будут использованы.

Параметры:
<ul>
<li><font color="#00aa00">dir</font> - директория, в которой будет создан pid-файл. Если не указано, то будет вызван File::Spec::tmpdir - который создаст файл в каком-нибудь каталоге /temp . Лучше указывать.</li>
<li><font color="#00aa00">name</font> - имя pid-файла.</li>
<li><font color="#00aa00">ext</font> - расширение для pid-файла. По умолчанию будет использоваться '.pid'.</li>
<li><font color="#00aa00">debug =&gt; 1</font> - включает режим отладки. Начнет выдавать дополнительную информацию через STDERR.</li>
<li><font color="#00aa00">quiet =&gt; 1</font> - включает "тихий" режим. Не выдает предупреждений об устаревших pid-файлах.</li>
</ul>

**abandon**

Указание не пытаться удалить файл в процессе завершения работы. Если <font color="#00aa00">abandon</font> не использовать, pid-файл будет удален после завершения работы скрипта.

## File::Pid

Попробовала использовать <font color="#00aa00">File::Pid</font> для блокировки запуска скрипта. Не получилось.

<pre>cpan install File::Pid</pre>

Пример, pid.pl:

```perl
use File::Pid;
  
my $pid = File::Pid->new({
  file => 'script.pid',
});
if ( my $num = $pid->running ) {
  die "Script already running: $num\n";
}

$pid->write;
$pid->remove;
```

Сразу же возникли проблемы:
<ul>
<li>если файл с pid еще не существует (а откуда ему взяться, при первом запуске скрипта) - выводится ошибка.
<pre># perl pid.pl
Can't kill a non-numeric process ID at 
/usr/local/share/perl/5.14.2/File/Pid.pm line 124.
</pre>
</li>
<li>если файл с pid создан, но сам pid в нем не записан - ошибка</li>
<li>если файл создан, и pid в нем указан, то pid текущего процесса в нем не сохраняется. Старый и уже не существующий pid продолжает сохраняться.
<pre># perl pid.pl</pre>
Содержимое script.pid:
<pre>33000</pre>
Реальный процесс:
<pre># ps -u root -f
root     31114 30877  0 01:11 pts/1    00:00:00 perl pid.pl
</pre>
</li>
</ul>

До проверки, действительно ли можно предотвратить повторный запуск скрипта с помощью
<font color="#00aa00">File::Pid</font> не дошло. Не вижу смысла. Возможно, данный модуль стал не актуальным с течением времени, написан с ошибками (что более вероятно), и имеет проблемы с использованием на разных операционных системах.

Вспомнила, что <font color="#00aa00">File::Pid</font> использовался в одной из систем, которую я встречала года 3 назад. Запуск скриптов там всегда был проблемой. Либо не запускались вообще, либо запускались в нескольких экземплярах. Теперь понятно - почему. Не рекомендую.

