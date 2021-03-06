﻿# CGI::Application. Основные методы приложений CGI::Application

*Частичный перевод <a href="http://search.cpan.org/~markstos/CGI-Application-4.31/lib/CGI/Application.pm">CGI::Application</a> / Essential Application Methods .
Как всегда - все примеры кода - авторские.*

Поскольку, я - поклонница Template Toolkit, и не вижу особого смысла использовать иные системы шаблонов,
если TT доступен, в данном документе опущено описание методов <font color="#00aa00">load_tmpl()</font> и
<font color="#00aa00">tmpl_path()</font>.

## param()

<pre>$self-&gt;param('step', 'on_get_result'); # устанавливаем значение параметру
my $step = $self-&gt;param('step'); # получаем значение параметра
</pre>
Метод <font color="#00aa00">param()</font> может использоваться для получения значений параметров, или для установки нужных значений определенным параметрам. Если установить значения параметров из метода <font color="#00aa00">setup</font> - эти значения станут доступны из любой части кода приложения, так же, как и самые обычные значения параметров.

Если вызвать <font color="#00aa00">param()</font> в контексте списка, метод вернет полный список всех существующих в данный момент параметров.
<pre>my @params = $self-&gt;param();</pre>

Не стоит путать этот метод с методом <font color="#00aa00">param()</font> модуля CGI.

## query()

<pre>my $q = $self-&gt;query();
my $day = $q-&gt;param('day');
</pre>
Метод <font color="#00aa00">query()</font> возвращает ссылку на объект query модуля CGI.pm . Сам объект создается при вызове метода <font color="#00aa00">new()</font> CGI::Application.

Используется метод <font color="#00aa00">query()</font>, как и обычно, для доступа к данным, которые передает клиент.

## run_modes()

<pre>$self-&gt;run_modes(
	on_get_result	=&gt; \&amp;on_get_result, # указываем ссылку на метод
	on_start	=&gt; 'on_start', # указываем имя метода
	AUTOLOAD	=&gt; sub { return 'Запрошенной страницы не существует' }
);
</pre>
Метод определяет таблицу соответствия возможных режимов (статусов) работы приложения и подпрограмм, которые выполняются, при запуске приложения в определенном режиме.

<font color="#00aa00">run_modes()</font> можно вызывать неограниченное число раз. Указанные режимы и подпрограммы будут добавлены к уже существующему списку. Если одному и тому же режиму назначаются в соответствие поочередно 2 и более разных подпрограмм - в итоге будет вызываться последний назначенный вариант.

Подготовленная методом <font color="#00aa00">run_modes()</font> таблица используется методом <font color="#00aa00">run()</font> для запуска требуемых функций и передачи им параметров запроса.

От назначенных методом <font color="#00aa00">run_modes()</font> подпрограмм ожидается, что они вернут текстовый блок информации (например, готовый код html-страницы), который без дополнительных преобразований будет отправлен клиенту.

## start_mode()

<pre>$self-&gt;start_mode('on_start');
</pre>
Метод <font color="#00aa00">start_mode()</font> назначает режим (один из определенных в списке <font color="#00aa00">run_modes()</font> ), который будет вызываться по-умолчанию, когда клиент не указал нужный режим в списке передаваемых параметров.

Если режим не назначен с помощью <font color="#00aa00">start_mode()</font>, CGI::Application в подобных случаях будет по-умолчанию вызывать режим "start".
