# Работа с документацией в Unix. Утилита man

*Небольшая заметка по теме мануалов. Где хранится документация в unix. Алгоритм работы утилиты man. Иерархия каталогов с документацией. Zcat и troff.*

Мануал - это руководство пользователя (от англ. user guide или user manual).

На примере Debian Linux.

## Где хранятся мануалы

Найти место хранения мануалов в вашей системе можно, используя название любой утилиты,
документацию на которую необходимой найти. Документация на каждую программу хранится в отдельном одноименном файле:
<pre>$ find / -name "alarm*"
/usr/lib/perl/5.14.2/auto/POSIX/alarm.al
/usr/share/doc/gawk/examples/prog/alarm.awk
/usr/share/man/man2/alarm.2.gz
</pre>
В данном случае, файл с расширением <font color="#00aa00">.2.gz</font> и будет содержать текст документации, а главной директорией, в которой хранятся все основные сведения о системе будет являться <font color="#00aa00">/usr/share/man</font> . В других Unix-системах, Solaris, MINIX и т.п., месторасположение каталога с документацией может немного отличаться. Теоретически, оно может отличаться даже в рамках одного бренда, но в разных версиях операционной системы. Нет ничего невозможного.

Содержимое главной директории с документацией:
<pre>$ ls /usr/share/man/
cs  de  fi  hu  it  ko    man2  man4  man6  man8  pl  pt_BR  sl  sv  zh_CN
da  es  fr  id  ja  man1  man3  man5  man7  nl    pt  ru     sr  tr  zh_TW
</pre>
Пример содержимого одного из подкаталогов:
<pre>$ ls /usr/share/man/ru/
man1  man5  man8
</pre>

## Иерархия мануалов

Справочная система unix включает в себя несколько разделов. Каждому разделу присвоен собственный номер:
<ol>
 	<li>Документация для исполняемых программ, утилит, описание команд оболочки (shell)</li>
 	<li>Описание системных вызовов системы (функции, предоставляемые ядром)</li>
 	<li>Документация по библиотечным вызовам (функции, предоставляемые программными библиотеками)</li>
 	<li>Специальные файлы (обычно находящиеся в каталоге /dev , например, "null", "pts", "tty" и др.)</li>
 	<li>Документация по форматам конфигурационных файлов, например о /etc/passwd</li>
 	<li>Игры (в моем случае, эта директория вообще была пустой)</li>
 	<li>Разное</li>
 	<li>Документация на команды администрирования системы (запустить которые можно только с правами суперпользователя)</li>
</ol>
Пример:
<pre>$ ls /usr/share/man/
cs  de  fi  hu  it  ko    man2  man4  man6  man8  pl  pt_BR  sl  sv  zh_CN
da  es  fr  id  ja  man1  man3  man5  man7  nl    pt  ru     sr  tr  zh_TW
</pre>
Описание для каждого системного вызова, каждой утилиты размещается в отдельном файле, упакованным в gz-архив:
<pre>$ ls /usr/share/man/man2/
accept.2.gz               getpriority.2.gz        outb_p.2.gz
accept4.2.gz              getresgid.2.gz          outl.2.gz
access.2.gz               getresgid32.2.gz        outl_p.2.gz
acct.2.gz                 getresuid.2.gz          outsb.2.gz
add_key.2.gz              getresuid32.2.gz        outsl.2.gz
adjtimex.2.gz             getrlimit.2.gz          outsw.2.gz
afs_syscall.2.gz          get_robust_list.2.gz    outw.2.gz
...
</pre>
&nbsp;
<h2>Как просмотреть документацию</h2>
Для того, чтобы просмотреть общее описание раздела, надо ввести команду:
<pre>$ man -S 6 intro</pre>

Аргумент <font color="#00aa00">-S</font> указывает номер раздела, откуда запрашивается информация.

Чтобы просмотреть документацию по определенной программе (утилите, системному вызову, функции), надо ввести команду:
<pre>$ man &lt;key&gt;
</pre>
где <font color="#00aa00">key</font> - это название утилиты.

В случае, когда существует одноименные утилиты и системные вызовы, следует указывать номер раздела, откуда вы хотите получить информацию. По-умолчанию, <font color="#00aa00">man</font> вернет вам ту страницу документации, которую первой найдет. Поиск будет осуществляться по всем доступным разделам, в заранее определенном порядке.

Например, вам надо получить информацию о <font color="#00aa00">sleep</font>.

**Вариант 1**

<pre>$ man -S 1 sleep
SLEEP(1)               User Commands                      SLEEP(1)

NAME
       sleep - delay for a specified amount of time

SYNOPSIS
       sleep NUMBER[SUFFIX]...
       sleep OPTION

DESCRIPTION
       Pause  for  NUMBER seconds.  SUFFIX may be `s' for seconds 
       (the default), `m' for minutes, `h' for hours or `d' for 
       days.  Unlike most implementations that require NUMBER be 
       an integer,  here  NUMBER  may  be  an  arbitrary floating 
       point number.  Given two or more arguments, pause for the 
       amount of time specified by the sum of their values.
...
</pre>
Обычно в заголовке страницы с документацией, пишется название команды, например, <font color="#00aa00">SLEEP(1)</font>. <font color="#00aa00">(1)</font> - обозначает номер раздела, где размещается найденная информация.

**Вариант 2**

<pre>$  man -S 3 sleep

SLEEP(3)        Linux Programmer's Manual                SLEEP(3)

NAME
       sleep - sleep for the specified number of seconds

SYNOPSIS
       #include &lt;unistd.h&gt;

       unsigned int sleep(unsigned int seconds);
...
</pre>

**Вариант 3**

<pre>$ man -S 2 sleep
Нет справочной страницы для sleep
Смотрите 'man 7 undocumented' в справке, если недоступны справочные страницы.
</pre>

## Как система находит эти мануалы

В Debian Linux существует конфигурационный файл <font color="#00aa00">/etc/manpath.config</font>, в котором прописаны пути, по которым система может искать документацию. При необходимости, можно вручную дописать в этот файл необходимые директории.

*/etc/manpath.config*:
<pre>
MANDATORY_MANPATH                       /usr/man
MANDATORY_MANPATH                       /usr/share/man
MANDATORY_MANPATH                       /usr/local/share/man

MANPATH_MAP     /bin                    /usr/share/man
MANPATH_MAP     /usr/bin                /usr/share/man
MANPATH_MAP     /usr/local/bin          /usr/local/man
MANPATH_MAP     /usr/local/bin          /usr/local/share/man
MANPATH_MAP     /usr/X11R6/bin          /usr/X11R6/man
...
</pre>
Примечание: Кроме того, в других операционных системах возможен вариант, когда необходимые пути прописаны в одной из переменных окружения.

Следует обратить внимание на утилиту <font color="#00aa00">manpath</font>, которая помогает определить пути поиска справочных страниц:
<pre>MANPATH(1)   Утилиты просмотра справочных страниц  MANPATH(1)

НАЗВАНИЕ
       manpath - определяет путь поиска справочных страниц

СИНТАКСИС
       manpath [-qgdchV] [-m система[,...]] [-C файл]

ОПИСАНИЕ
       Если  установлена переменная окружения $MANPATH, то 
       manpath просто покажет её значение и выдаст 
       предупреждение. Если нет, то manpath определит 
       подходящий путь поиска иерархии справочных страниц и 
       отобразит результаты.

       Список путей поиска (через двоеточие) определяется по 
       данным файла  настройки  man-db  (/etc/manpath.config)
       и пользовательского окружения.
...
</pre>
Пример вывода утилиты manpath:
<pre>$ manpath
/usr/local/man:/usr/local/share/man:/usr/share/man
</pre>

## Как работает утилита man

Утилита <font color="#00aa00">man</font> в unix-системах предназначена для комфортного просмотра документации.

Алгоритм работы утилиты - это конвеер, последовательное выполнение нескольких операций:
<ul>
 	<li>поиск файла с нужной страницей документации;</li>
 	<li>чтение и распаковка файла;</li>
 	<li>форматирование страницы;</li>
 	<li>вывод отформатированного текста для просмотра.</li>
</ul>
Описание каждой утилиты или системного вызова хранится в отдельном файле. В Debian Linux, каждая страница документации упакована в gzip-архив.

**Первый этап** работы man-утилиты - это поиск страницы с указанным именем последовательно во всех директориях с документацией.

**На втором этапе** необходимо распаковать файл. В принципе, для лучшего понимания, как работает утилита, можно распаковать файл документации вручную, и посмотреть его содержимое:
<pre>$ zcat /usr/share/man/man2/alarm.2.gz
...
.TH ALARM 2 2008-06-12 "Linux" "Linux Programmer's Manual"
.SH NAME
alarm \- set an alarm clock for delivery of a signal
.SH SYNOPSIS
.nf
.B #include <unistd.h>
...
</unistd.h></pre>
<font color="#00aa00">zcat</font> - это удобная утилита, которая позволяет распаковать файл в <font color="#00aa00">gzip</font> формате и вывести результат в <font color="#00aa00">stdout</font>. В результате, мы видим кусок файла, для разметки которого использовался специальный формат.

**Третий этап** - форматирование файла. Для хранения документации в unix используется старый формат - <font color="#00aa00">ROFF</font>. Он позволяет сделать разметку текста для представления в виде документа. Первые версии ROFF формата появились в 60-х годах прошлого столетия. Для чтения документа в ROFF-формате, можно использовать инструменты <font color="#00aa00">troff</font>, <font color="#00aa00">nroff</font> и <font color="#00aa00">groff</font>.
<pre>$ zcat /usr/share/man/man2/alarm.2.gz | troff -a
&lt;beginning of="" page=""&gt;
alarm &lt;-&gt; set an alarm clock for delivery of a signal
arranges for a signal to be delivered to the calling process in seconds.
...
&lt;/beginning&gt;</pre>
**На последнем этапе**, полученный текст передается на вход утилите <font color="#00aa00">more</font> или <font color="#00aa00">less</font>. Конкретный выбор зависит от настроек операционной системы.

Та же самая страница при просмотре через <font color="#00aa00">man</font>:
<pre>$ man -S 2 alarm
ALARM(2)     Linux Programmer's Manual            ALARM(2)

NAME
       alarm - set an alarm clock for delivery of a signal

SYNOPSIS
       #include &lt;unistd.h&gt;
...
</pre>

