# Использование статики в Catalyst-приложении (плагин Static::Simple)

Мое тестовое catalyst-приложение начинает потихоньку развиваться и приобретать функциональность. Для создания интерфейса администратора я решила использовать <font color="#00aa00">jQuery UI</font> и мне потребовалось добавить в шаблоны ссылки на статические файлы. Ссылки были добавлены, каждую из них catalyst попытался обработать и вернул 404.

Решив, что с этой проблемой может столкнуться любой начинающий разработчик, решила написать небольшую заметку о том, как заставить catalyst адекватно обрабатывать статические файлы.

Приведенный код строится на ранее созданной основе: <a href="Catalyst_intro.md">Как создать catalyst-приложение с нуля</a>

**1.** Для работы со статикой надо подключить модуль <font color="#00aa00">Static::Simple</font> и выполнить настройки.

*/lib/app.pm* :

```perl
use Catalyst qw/
    Static::Simple
/;

__PACKAGE__-&gt;config(
    'Plugin::Static::Simple' =&gt; {
        include_path =&gt; [
            '/static',
            app-&gt;config-&gt;{root},
        ],
        ignore_dirs =&gt; [ qw/images css js help/ ],
    }
);

__PACKAGE__-&gt;setup();
```

<ul>
<li><b>include_path</b> - <font color="#00aa00">include_path</font> позволяет задать список путей, где catalyst-приложение будет искать ваши статические файлы. Для того, чтобы добавить корневой каталог, надо использовать <font color="#00aa00">MyApp-&gt;config-&gt;{root}</font>.</li>
<li><b>ignore_extensions</b> - По-умолчанию, типы файлов tmpl, tt, tt2, html и xhtml - статическими не считаются, независимо от того - в какой директории они размещены. Попытка обратиться к ним, в рамках статических директорий, вызовет ошибку:
<pre>Static::Simple: Ignoring extension `html`</pre>
Чтобы изменить поведение <font color="#00aa00">Static::Simple</font> и позволить catalyst-приложению использовать html-файлы, например, для организации статических страниц руководства пользователя, можно переопределить список игнорируемых расширений с помощью опции <font color="#00aa00">ignore_extensions</font>. Либо просто использовать другие расширения файлов, например, *.htm вместо *.html.</li>

<li><b>ignore_dirs</b> - <font color="#00aa00">ignore_dirs</font> подсказывает catalyst, какие директории содержат статику и не подлежат специальной обработке. При этом, не важно где размещается указанная директория. Если указано: <font color="#00aa00">ignore_dirs =&gt; [ qw/images/ ]</font>,
<pre>/root/static/images
/root/images
/root/src/images 
</pre>
- все файлы этих директорий будут обрабатываться, как статичные</li>

<li><b>expires</b> - Использование <font color="#00aa00">expires</font> приводит к тому, что в заголовках http-ответа будет возвращаться поле <font color="#00aa00">Expires</font>. Значение поля будет зависеть от того, какое количество секунд было указано в параметре <font color="#00aa00">expires</font>.</li>
</ul>

Более подробный список опций для конфигурирования <font color="#00aa00">Static::Simple</font> приведены в официальной документации, см. раздел "Полезные ссылки"

**2.** Интерфейс администратора, для работы с логами, будет доступен в специальной директории. Создаем специальный контроллер.

*/lib/app/Controller/Log.pm* :

```perl
package app::Controller::Log;

use uni::perl ':dumper';
use Moose;
use namespace::autoclean;

BEGIN { extends 'Catalyst::Controller' }

sub index :Path('/log') :Args(0) {
    my ( $self, $c ) = @_;

    $c-&gt;stash-&gt;{template} = 'log/index.tt';
}

__PACKAGE__-&gt;meta-&gt;make_immutable;

1;
```

**3.** Создаем шаблон для вывода главной страницы интерфейса администратора. Для начала, выведем по центру картинку. Картинка лежит по адресу <font color="#00aa00">/root/static/images/catalyst_logo.png</font>.

*/root/src/log/index.tt* :

```html
TEXT
<img src="/static/images/catalyst_logo.png">
```

После этого запускаем сервер и заходим на страницу <font color="#00aa00">http://localhost:3000/log</font> .

**4.** Можно добавить "руководство пользователя", раздел помощи на сайте, разместив статические файлы в catalyst-приложении.

*/root/static/help/index.htm* :

```html
<b>Hello!</b>
```

После запуска сервера этот файл будет доступен по адресу <font color="#00aa00">http://localhost:3000/static/help/index.htm</font>

