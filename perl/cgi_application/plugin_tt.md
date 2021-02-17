# CGI::Application и Template Toolkit. Использование плагина CGI::Application::Plugin::TT

*Публикация - в значительной мере - перевод официальной документации CGI::Application::Plugin::TT.
Все примеры - оригинальные, разработанные специально для данного документа и проверенные на работоспособность.*

CGI::Application поддерживает работу с шаблонами Template Toolkit с помощью CGI::Application::Plugin::TT .

## Методы

### tt_process

<font color="#00aa00">tt_process()</font> - это "обертка" вокруг стандартного метода <font color="#00aa00">process()</font> от Template Toolkit. Использование <font color="#00aa00">tt_process()</font> похоже на использование <font color="#00aa00">process()</font>. Методу <font color="#00aa00">tt_process</font> можно передать до двух параметров. Первый - это имя вызываемого шаблона, второй - ссылка на хэш с передаваемыми в шаблон данными. Оба параметра являются не обязательными.

Если не указано имя шаблона, имя будет сгенерировано автоматически, с помощью функции <font color="#00aa00">$self-&gt;tt_template_name</font> .
<pre>...
my $tt_params = {};
$tt_params-&gt;{'block'} = 'show_result';

return $self-&gt;tt_process('template.tt', $tt_params);
...
</pre>
<font color="#00aa00">tt_process()</font> возвращает скалярную ссылку на готовый к выводу пользователю код шаблона.

Пример:
<pre>...
my $scalar_link = $self-&gt;tt_process('template.tt', $tt_params);
print ${$scalar_link};
...
</pre>

Вывод:

```html
<meta http-equiv="Content-Type" content=" text/html;
charset=windows-1251">

<title>Нумерология...</title>

<style type="text/css">
body {font-family: Verdana, Arial Cyr, Arial, Helvetica; 
font-weight: normal; font-size: 9pt;}

</style>

```

Фактически, полученный код не обязательно отправлять на вывод пользователю. Можно сохранить данные в файл, или разместить и отправить в электронном письме.

### tt_config

Метод <font color="#00aa00">tt_config()</font> позволяет задать некоторые параметры работы плагина CGI::Application::Plugin::TT и Template Toolkit. Рекомендуется вызывать метод <font color="#00aa00">tt_config()</font> до того, как будут выполнены первые обращения к методам <font color="#00aa00">tt_process()</font> и <font color="#00aa00">tt_obj()</font>. Если хотите получить ошибку - сделайте наоборот.

#### TEMPLATE_OPTIONS

<font color="#00aa00">TEMPLATE_OPTIONS</font> позволяет определить параметры для объекта Template. В списке можно указывать все классические конфигурационные опции Template Toolkit.
<pre>package App2;
...
sub setup {
  my $self = shift;
  $self-&gt;start_mode('on_start');	

  $self-&gt;tt_config(
    TEMPLATE_OPTIONS =&gt; {
    INCLUDE_PATH =&gt; '/home/aninatalie.ru/templates',
    },
  );

  $self-&gt;run_modes(
    on_start	=&gt; \&amp;on_start,
    AUTOLOAD	=&gt; sub { return 'Запрошенной страницы не существует' }
  );

}
...
</pre>

#### TEMPLATE_NAME_GENERATOR

<font color="#00aa00">TEMPLATE_NAME_GENERATOR</font> позволяет указать ссылку на функцию, которая будет генерировать имя файла шаблона. Данная функция будет выполняться, если метод <font color="#00aa00">tt_process()</font> будет вызван без указания имени шаблона.

Если специальная функция не назначена, а <font color="#00aa00">tt_process()</font> все-таки вызван без параметров - по-умолчанию будет сгенерировано имя шаблона на основе имен текущего модуля и метода.
<pre>package App2;
...
sub setup {
	my $self = shift;

	$self-&gt;mode_param('step');
	$self-&gt;start_mode('on_start');	

	$self-&gt;tt_config(
		TEMPLATE_NAME_GENERATOR =&gt; \&amp;my_own_generator,
	);

	$self-&gt;run_modes(
		on_start	=&gt; \&amp;on_start,
		AUTOLOAD	=&gt; sub { return 'Запрошенной страницы 
		не существует' }
	);
}

sub my_own_generator {
	my $self = shift;	
	return 'error.tt',
}

sub on_start {
	my $self = shift;

	return $self-&gt;tt_process();
}
</pre>

### tt_obj

Метод <font color="#00aa00">tt_obj()</font> возвращает ссылку на основной объект Template Toolkit. Используется, в основном, в процессе отладки приложения.

### tt_params

Метод <font color="#00aa00">tt_params()</font> определяет список параметров, которые будут
передаваться в каждый вызываемый шаблон.

Как правило, при вызове метода <font color="#00aa00">tt_process()</font>, ему передается ссылка
на хэш с параметрами. Параметры, определенные с помощью <font color="#00aa00">tt_params()</font> будут добавляться в этот хэш.
<pre>sub setup {
	my $self = shift;

	$self-&gt;mode_param('step');
	$self-&gt;start_mode('on_start');

	$self-&gt;tt_params(block =&gt; 'secret_page');

	$self-&gt;run_modes(
		on_start	=&gt; \&amp;on_start,
		AUTOLOAD	=&gt; sub { return 'Запрошенной страницы
		не существует' }
	);
}
sub on_start {
	my $self = shift;
	my $q = $self-&gt;query();
	my $tt_params = {}; # к значениям данного хэша будут добавлены 
			# значения, заданные в tt_params()
	...
	return $self-&gt;tt_process('template2.tt', $tt_params);
}
</pre>
Если один и тот же параметр определен дважды - преимуществом будет пользоваться
параметр, который входит в хэш, передаваемый непосредственно <font color="#00aa00">tt_process()</font>.

Пример:
<pre>sub setup {
	my $self = shift;
	...
	$self-&gt;tt_params(block =&gt; 'show_result');
}

sub start {
	my $self = shift;
	my $tt_params = {
		block =&gt; 'show_form',
	};

	return $self-&gt;tt_process('template.tt', $tt_params);
}
</pre>

В результате выполнения данного блока кода, в шаблоне будет использоваться параметр block со значением: "show_form".

### tt_clear_params

Метод <font color="#00aa00">tt_clear_params()</font> удалит все сохранившиеся до текущего времени параметры, установленные методом <font color="#00aa00">tt_params()</font>.

<pre>$self-&gt;tt_clear_params;</pre>

### tt_pre_process

Метод <font color="#00aa00">tt_pre_process()</font> действует аналогично методу <font color="#00aa00">cgiapp_prerun()</font>. <font color="#00aa00">tt_pre_process()</font> вызывается и отрабатывается непосредственно перед началом обработки шаблона.

Выполнение <font color="#00aa00">tt_pre_process()</font> будет запущено автоматически, а на вход будут переданы те же параметры, которые передаются методу <font color="#00aa00">tt_process()</font>. При желании, можно внести последние изменения в список передаваемых шаблону параметров.
<pre>sub tt_pre_process {
	my ($self, $file, $vars) = @_;
	$vars-&gt;{block}='show_result';
	print STDERR "start template process: $file \n";
	return;
}
</pre>

### tt_post_process

Метод <font color="#00aa00">tt_post_process()</font> аналогичен методу <font color="#00aa00">cgiapp_postrun()</font>. Вызывается автоматически после завершения обработки шаблона, но до отправления результата клиенту. На вход принимает скалярную ссылку на обработанный шаблон - готовый к выводу html-код.
<pre>sub tt_post_process {
	my ($self, $htmlref) = @_;

	print $htmlref;

	return;
}
</pre>

### tt_template_name

Метод генерирует название шаблона на основе имени вызывающих его класса и метода.

Например:
<pre>package App;

use base 'CGI::Application';
use CGI::Application::Plugin::TT;
 
sub setup {
	my $self = shift;

	$self-&gt;run_modes(
		on_start =&gt; 'start',
		...
	);
}
...
sub start {
	my $self = shift;

	my $template_name = $self-&gt;tt_template_name();
	return $self-&gt;tt_process($template_name);
}
</pre>
Результат: Для вывода будет запрошен шаблон App/start.tmpl .

<font color="#00aa00">tt_template_name()</font> позволяет формировать и использовать определенную иерархию шаблонов, зависимую от структуры самого приложения.

Можно заставить <font color="#00aa00">tt_template_name()</font> сгенерировать название шаблона на основе не вызывающего метода, а вышестоящего по уровню. На сколько уровней выше - задается с помощью
числа в качестве параметра.

Пример:
<pre>my $template_name = $self-&gt;tt_template_name(1);
</pre>
Результат в файле error.log:
<pre>[Wed Jun  9 15:56:55 2010] [error] Error executing run mode 
'on_start': file error - App/(eval).tmpl: not found 
at /home/aninatalie/aninatalie.ru/cgi-bin/app.cgi line 8\n
</pre>
<font color="#00aa00">tt_template_name()</font> вызывается автоматически, если при вызове метода <font color="#00aa00">tt_process()</font> не было указано имя шаблона.

### tt_include_path

Метод <font color="#00aa00">tt_include_path()</font> позволяет указать путь к месторасположению шаблонов. Причем, на этапе, когда объект TT уже существует. Обычно для указания пути используют INCLUDE_PATH.

Вызов <font color="#00aa00">tt_include_path()</font> внесет изменения в INCLUDE_PATH, что можно использовать во всех последующих вызовах объекта TT.

<pre>my $dir = '/home/aninatalie/aninatalie.ru/templates';
$self-&gt;tt_include_path( [$dir] );
</pre>


