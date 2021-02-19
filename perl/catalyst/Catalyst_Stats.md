# Catalyst::Stats

Catalyst::Stats - класс Catalyst, работа со статистикой, измерение времени выполнения отдельных экшенов.

Модуль Catalyst::Stats используется по-умолчанию, для вывода информации в логе Сatalyst-приложения. Если вы хотите заменить стандартный модуль чем-то своим, понадобится внести изменения в конфиг MyApp.pm:

```perl
__PACKAGE__->stats_class( "My::Stats" );
```
Кроме того, ваш модуль должен будет реализовать те же самые методы, что и Catalyst::Stats.


## Синтаксис

```perl
$stats = $c->stats;
$stats->enable(1);
$stats->profile($comment);
$stats->profile(begin => $block_name, comment =>$comment);
$stats->profile(end => $block_name);
$elapsed = $stats->elapsed;
$report = $stats->report;
```

Catalyst использует Catalyst::Stats чтобы информировать вас о времени выполнения экшенов. Вы можете добавить собственные контрольные точки в код приложения, для более подробной отчетности и контроля.

**Пример:**

Информация в логе при обращении по адресу http://localhost:3000/support до внесения изменений в код экшена, стандартный вывод:
<pre>
[info] Request took 0.123658s (8.087/s)
.------------------------------------------------------------+-----------.
| Action                                                     | Time      |
+------------------------------------------------------------+-----------+
| /support/index                                             | 0.116085s |
|  -&gt; MyApp::View::TT-&gt;process                               | 0.110125s |
| /end                                                       | 0.000368s |
'------------------------------------------------------------+-----------'
</pre>

Теперь добавим в код экшена вызов метода profile():

```perl
sub index :Path :Args(0) {
    my ( $self, $c ) = @_;

    $c->stats->profile(begin => "index");
    $c->stats->profile("starting critical bit");

    $c->stash->{template} = 'support/index.tt';
    $c->stash->{message}  = 'Hello World!';

    $c->stats->profile("completed critical bit");
    $c->stats->profile(end => "index");

    $c->forward('View::TT');
}
```

Catalyst::Stats - используется Catalyst-приложением по умолчанию, поэтому, добавляя дополнительные инструкции в экшен, нет необходимости указывать в начале файла код подключения модуля, или дополнительные параметры конфигурации, или т.п.

Данные в логе после внесения изменений и выполнения запроса к странице *http://localhost:3000/support* :
<pre>
[info] Request took 0.160763s (6.220/s)
.------------------------------------------------------------+-----------.
| Action                                                     | Time      |
+------------------------------------------------------------+-----------+
| /support/index                                             | 0.144864s |
|  index                                                     | 0.000684s |
|   - starting critical bit                                  | 0.000215s |
|   - completed critical bit                                 | 0.000263s |
|  -&gt; MyApp::View::TT-&gt;process                               | 0.142974s |
| /end                                                       | 0.000369s |
'------------------------------------------------------------+-----------'
</pre>

## Методы Catalyst::Stats

**new**

Создает новый объект Catalyst::Stats.

```perl
$stats = Catalyst::Stats->new;
```

**enable**

```perl
$stats->enable(0);
$stats->enable(1);
```

Отключает или включает вывод статистики. По умолчанию обычно включен. Т.е. если вы добавите в код $stats-&gt;enable(0) :

```perl
sub index :Path :Args(0) {
    my ( $self, $c ) = @_;

    $c->stats->enable(0);

    $c->stash->{template} = 'support/index.tt';
    $c->stash->{message}  = 'Hello World!';
    $c->forward('View::TT');
```

То вся статистика с момента отработки этого кода, перестанет выводиться:
<pre>
[info] Request took 0.005904s (169.377/s)
.------------------------------------------------------------+-----------.
| Action                                                     | Time      |
+------------------------------------------------------------+-----------+
| /support/index                                             | 0.002187s |
'------------------------------------------------------------+-----------'
</pre>

**profile**

```perl
$stats->profile($comment);
$stats->profile(begin => $block_name, comment =>$comment);
$stats->profile(end => $block_name);
```

Устанавливает контрольные точки для профайлинга. Можно использовать в паре begin/end, и тогда он будет измерять время выполнения от того места, где установлен begin. Либо можно устанавливать контрольные точки без begin/end, и тогда в статистике будет выводиться время, прошедшее с момента последнего вывода статистики в лог (либо это был вывод связанный с началом текущего экшена, либо предыдущей контрольной точки).

```perl
sub index :Path :Args(0) {
    my ( $self, $c ) = @_;

    $c->stats->profile("starting critical bit");

    $c->stash->{template} = 'support/index.tt';
    $c->stash->{message}  = 'Hello World!';
    $c->stats->profile("completed critical bit");
    $c->forward('View::TT');
}
```

<pre>[info] Request took 0.150029s (6.665/s)
.------------------------------------------------------------+-----------.
| Action                                                     | Time      |
+------------------------------------------------------------+-----------+
| /support/index                                             | 0.137463s |
|  - starting critical bit                                   | 0.000325s |
|  - completed critical bit                                  | 0.000237s |
|  -&gt; MyApp::View::TT-&gt;process                               | 0.135906s |
| /end                                                       | 0.000380s |
'------------------------------------------------------------+-----------'
</pre>

Следующие аргументы - ключ/значение могут использоваться в методе profile():

<ul>
<li>begin =&gt; ACTION - Помечает начало контрольного блока. Этот блок в статистике будет отмечен как отдельный "подраздел".</li>
<li>end =&gt; ACTION - Помечает конец контрольного блока. Используется в паре с меткой begin.</li>
<li>comment =&gt; COMMENT - Помечает очередную контрольную точку и выводит ее в статистике с указанным комментарием. Выражение
<pre>
$c->stats->profile(comment => "COMMENT"); 
</pre>
идентично
<pre>$c->stats->profile("COMMENT");
</pre>
</li>
 	<li>uid =&gt; UID - Устанавливает предопределенный уникальный ID. Может оказаться полезным, для случаях сложной логики профилирования.</li>
 	<li>parent =&gt; UID - Связь контрольной точки с ранее установленным ID.</li>
</ul>

**created**

```perl
($seconds, $microseconds) = $stats->created;
```

Возвращает время создания объекта, в gettimeofday формате, в секундах и микросекундах.

**elapsed**

```perl
$elapsed = $stats->elapsed
```

Возвращает время, которое прошло с момента создания объекта Catalyst::Stats, в секундах.

**report**

```perl
print $stats->report ."\n";
$report = $stats->report;
@report = $stats->report;
```

Метод при вызове в скалярном контексте возвращает текстовый отчет. По сути, это те же данные, какие выводятся в конце выполнения каждого запроса в логе, но в данном случае, в отчете содержатся данные только по тем экшенам и контрольным точкам, которые успели выполниться до того момента, когда был вызван метод report():

```perl
my $report = $c->stats->report;
$c->log->debug(Dumper($report));
```

<pre>[debug] $VAR1 = 
'.----------------------------------------------------+-----------.
| Action                                              | Time      |
+-----------------------------------------------------+-----------+
| /support/index                                      | 0.007089s |
|  - starting critical bit                            | 0.000313s |
|  - completed critical bit                           | 0.000253s |
\'----------------------------------------------------+-----------\'
';
</pre>
При вызове в контексте массива, метод возвращает сложную структуру данных. Каждая строка структуры содержит данные:

<pre>[ depth, description, time, rollup ].</pre>

```perl
my @report = $c->stats->report;
$c->log->debug(Dumper(@report));
```

Результат выполнения данного кода:
<pre>
[debug] $VAR1 = [
          0,
          '/support/index',
          '0.0072',
          1
        ];

$VAR2 = [
          1,
          '- starting critical bit',
          '0.00032',
          0
        ];

$VAR3 = [
          1,
          '- completed critical bit',
          '0.00025',
          0
        ];
</pre>

## Совместимые методы

Некоторые компоненты ожидают, что для работой со статистикой используется не Catalyst::Stats, а Tree::Simple, и они рассчитывают, что будут обращаться к объекту Tree::Simple. Для совместимости в Catalyst::Stats добавлены методы:
<ul>
<li><b>accept</b></li>
<li><b>addChild</b></li>
<li><b>setNodeValue</b></li>
<li><b>getNodeValue</b></li>
<li><b>traverse</b></li>
</ul>
