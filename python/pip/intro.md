# Работа с pip для начинающих. Шпаргалка

## Установка pip

**pip** - питоняшный менеджер пакетов. Устанавливает и удаляет пакеты, написанные с помощью python.
<font color="#00aa00"><pre>$ apt-get install python-pip
$ pip install --upgrade pip
</pre></font>
Проверяем, что установка прошла успешно:
<font color="#00aa00"><pre>$ pip -V
pip 18.0 from /usr/local/lib/python2.7/dist-packages/pip (python 2.7)
</pre></font>
Pip по умолчанию устанавливает пакеты из Python Package Index (PyPI) - крупнейшего каталога пакетов для python. Python Package Index - аналог CPAN для Perl и PEAR для PHP.

## Установка pip для версии Python 3.5 на Debian Linux

<font color="#00aa00"><pre>$ apt-get install curl
$ curl https://bootstrap.pypa.io/ez_setup.py -o - | python3.5

$ pip3 --version
pip 19.0.2 from /usr/local/lib/python3.5/dist-packages/pip-19.0.2-py3.5.egg/pip
 (python 3.5)
</pre></font>
Второй вариант установки pip для Python 3.5:
<font color="#00aa00"><pre>$ apt-get install python3-pip</pre></font>
Результат:
<font color="#00aa00"><pre>$ pip3 -V
pip 9.0.1 from /usr/lib/python3/dist-packages (python 3.5)
</pre></font>

## Страница помощи pip

Команда **pip help** - выводит список основных команд и опций pip. Можно использовать опцию *--help* для получения справки по конкретным командам pip.

Пример:
<font color="#00aa00"><pre>$ pip install --help
Usage:
  pip install [options] &lt;requirement specifier=""&gt; [package-index-options] ...
  pip install [options] -r &lt;requirements file=""&gt; [package-index-options] ...
...
&lt;/requirements&gt;&lt;/requirement&gt;</pre></font>

## Как установить новый пакет или обновить ранее установленный

**pip install package_name** - установка нового пакета или обновление ранее установленного.

*Полезные опции:*

*--help* - подробная справка по команде install и ее опциям.

*-r*, *--requirement &lt;file&gt;* - установка пакетов по списку из указанного requirement-файла.
*-U*, *--upgrade* - обновление пакета до последней доступной версии.
*-I*, *--ignore-installed* - игнорировать уже установленные пакеты, не переустанавливать и не обновлять.
*-i*, *--index-url &lt;url&gt;* - ссылка на Python Package Index (по умолчанию - https://pypi.org/simple).
*--force-reinstall* - принудительная переустановка пакета.
*--target &lt;dir&gt;* - установка пакетов в указанную директорию.
*--no-deps* - установка пакета без зависимостей.
*--user* - установка пакета в директорию юзера.
*--compile* - компилировать файлы в байт-код.
*--no-compile* - не компилировать в байт-код.
*--extra-index-url &lt;url&gt;* - дополнительные адреса индексов пакетов.

Примеры:
<font color="#00aa00"><pre>$ pip install --upgrade MySQL-python
$ pip install -U --force-reinstall django
$ pip install django --user
$ pip install tornado --upgrade --ignore-installed
$ pip install tornado==5.1
</pre></font>

## Как удалить установленный пакет

**pip uninstall package_name**

*Полезные опции:*

*-y*, *--yes* - pip не будет запрашивать подтверждений, автоматическое согласие на все действия.
*-r*, *--requirement &lt;file&gt;* - удалить все пакеты, перечисленные в специальном requirement-файле.

Пример:
<font color="#00aa00"><pre>$ pip uninstall MySQL-python</pre></font>

## Как скачать пакет

**pip download** позволяет скачивать пакеты из PyPI или проекты из отдельно указанных систем контроля версий. Можно осуществлять загрузку списка пакетов из указанного файла *requirements.txt*.
<font color="#00aa00"><pre>$ pip download AsyncIO
...
Successfully downloaded AsyncIO
$
$ ls
asyncio-3.4.3.tar.gz
</pre></font>

## Вычисление хэша для архива с помощью pip

Команда **pip hash** вычисляет хэш для указанного файла.
<font color="#00aa00"><pre>$ pip hash asyncio-3.4.3.tar.gz
asyncio-3.4.3.tar.gz:
--hash=sha256:83360ff8bc97980e4ff25c964c7bd3923d333d177aa4f7fb736b019f26c7cb41
</pre></font>
*Полезная опция:*

*-a*, *--algorithm &lt;algorithm&gt;* - позволяет указать "sha256", "sha384" или "sha512", чтобы выбрать предпочтительный алгоритм вычисления хэш-суммы.

Чтобы проверить, работает ли **pip hash** только с архивами пакетов python, я создала простой текстовый файл, который содержит внутри несколько чисел и скормила его **pip hash**:
<font color="#00aa00"><pre>$ pip hash 1.txt
1.txt:
--hash=sha256:03c5ab10d240d4ec124c471e0501b5ffecd45f5269b1a77fb687e1c127c1387c
</pre></font>
Затем вычислила хэш-сумму с помощью shasum:
<font color="#00aa00"><pre>apache@debian:~$ shasum -a 256 1.txt
03c5ab10d240d4ec124c471e0501b5ffecd45f5269b1a77fb687e1c127c1387c  1.txt
</pre></font>
Результат идентичен.

## Создание файла requirements.txt для проекта с помощью pip

В мире python достаточно часто используются файлы *requirements.txt*, которые содержат список всех необходимых пакетов для вашего приложения.

*requirements.txt* полезен как с точки зрения документирования проекта, так и для автоматической установки необходимого проекту окружения - пакетов со всеми их зависимостями.

Файл *requirements.txt* обычно размещают в корне проекта. Можно, по мере написания программного кода проекта, формировать файл вручную, либо - с помощью pip и команды **freeze**.

Команда **pip freeze** выводит список всех установленных пакетов в специальном формате:
<font color="#00aa00"><pre>$ pip freeze
CDDB==1.4
Django==1.11.2
MySQL-python==1.2.5
Pillow==2.6.1
Pygments==2.0.1
SOAPpy==0.12.22
argparse==1.2.1
cffi==0.8.6
chardet==2.3.0
...
</pre></font>
*requirements.txt* можно получить, просто перенаправив вывод в файл:
<font color="#00aa00"><pre>$ pip freeze &gt; requirements.txt
</pre></font>
В дальнейшем, данный список можно использовать для установки всех необходимых пакетов в новом окружении:
<font color="#00aa00"><pre>$ pip install -r requirements.txt
</pre></font>
*Полезные опции:*

*-l*, *--local* - в список пакетов будут включены только пакеты виртуального окружения (virtualenv).
*--user* - в список пакетов будут включены только пакеты установленные в пространстве текущего пользователя.

## Как просмотреть список установленных пакетов

Команда **pip list** выводит список всех установленных пакетов в удобной для чтения табличной форме.

*Полезные опции:*

*-o*, *--outdated* - список устаревших пакетов, для которых доступны обновления.
*-u*, *--uptodate* - список пакетов не требующих обновления.
*-l*, *--local* - только список пакетов виртуального окружения (virtualenv).
*--user* - только список пакетов установленных в окружении пользователя.

Пример, список устаревших пакетов:
<font color="#00aa00"><pre>$ pip list --outdated
Package          Version  Latest   Type
cffi             0.8.6    1.11.5   wheel
chardet          2.3.0    3.0.4    wheel
colorama         0.3.2    0.3.9    wheel
cryptography     0.6.1    2.3      wheel
feedparser       5.1.3    5.2.1    sdist
...
</pre></font>
Пример, список пакетов не требующих обновления:
<font color="#00aa00"><pre>$ pip list --uptodate
Package      Version
MySQL-python 1.2.5
pip          18.0
six          1.11.0
SOAPpy       0.12.22
</pre></font>

## Как получить информацию об установленном пакете

**pip show package_name** - выводит информацию об установленном пакете (версия, месторасположение на диске, зависимости, домашняя страница и т.д.).

*Полезная опция:*

*-f*, *--files* - добавляет к основной информации полный список установленных файлов указанного пакета.

Пример:
<font color="#00aa00"><pre>$ pip show tornado --files
Name: tornado
Version: 5.1
Summary: Tornado is a Python web framework and asynchronous networking library, 
originally developed at FriendFeed.
Home-page: http://www.tornadoweb.org/
Author: Facebook
Author-email: python-tornado@googlegroups.com
License: http://www.apache.org/licenses/LICENSE-2.0
Location: /usr/local/lib/python2.7/dist-packages
Requires: backports-abc, singledispatch, futures
Required-by:
Files:
  tornado-5.1.dist-info/DESCRIPTION.rst
  tornado-5.1.dist-info/INSTALLER
  tornado-5.1.dist-info/METADATA
  tornado-5.1.dist-info/RECORD
  tornado-5.1.dist-info/WHEEL
  tornado-5.1.dist-info/metadata.json
  tornado-5.1.dist-info/top_level.txt
...
</pre></font>

## Как найти нужный пакет

**pip search** -  поиск пакета по имени пакета или его описанию.

Пример. Список всех пакетов, в названии или описании которых есть pyopenssl:
<font color="#00aa00"><pre>$ pip search pyopenssl
egenix-pyopenssl (0.13.16)  - eGenix pyOpenSSL Distribution for Python
pyOpenSSL (18.0.0)          - Python wrapper module around the OpenSSL library
  INSTALLED: 0.14
  LATEST:    18.0.0
gevent_openssl (1.2)        - A gevent wrapper for pyOpenSSL
service_identity (17.0.0)   - Service identity verification for pyOpenSSL.
ndg-httpsclient (0.5.1)     - Provides enhanced HTTPS support for httplib and
urllib2 using PyOpenSSL
  INSTALLED: 0.3.2
  LATEST:    0.5.1
ssl_sni (0.1)               - A wrapper to pyOpenSSL to provide an interface like
the standard ssl module.
backports.ssl (0.0.9)       - The Python 3.4 standard `ssl` module API implemented
on top of pyOpenSSL
</pre></font>

## Простая проверка на совместимость

**pip check** - проверка зависимостей установленных пакетов на совместимость между собой.
<font color="#00aa00"><pre>$ pip check
No broken requirements found.
</pre></font>

## Как настроить автодополнение в pip

В большинстве Linux систем при наборе команд в командной строке можно пользоваться
автодополнением. После набора первых нескольких символов имени файла и нажатия клавиши [TAB], система автоматически допишет имя или, если существует несколько вариантов продолжения строки, выведет список всех доступных версий.

В pip тоже можно настроить нечто подобное, используя команду - **completion**.
<font color="#00aa00"><pre>$ pip completion --bash &gt;&gt; ~/.bashrc
</pre></font>
В файл *.bashrc* будут добавлены строки:
<font color="#00aa00"><pre>$ pip bash completion start
_pip_completion()
{
    COMPREPLY=( $( COMP_WORDS="${COMP_WORDS[*]}" \
                   COMP_CWORD=$COMP_CWORD \
                   PIP_AUTO_COMPLETE=1 $1 ) )
}
complete -o default -F _pip_completion pip
$ pip bash completion end
</pre></font>
Если вы внесли правки в *.bashrc* пользователя USER1, то автодополнение будет работать именно для этого пользователя, не для всех остальных.

Перезагрузить *.bashrc* без выхода из системы можно с помощью команд:
<font color="#00aa00"><pre>$ . ~/.bashrc</pre></font>
или
<font color="#00aa00"><pre>$ exec bash</pre></font>
Теперь можно проверить работу автодополнения. У меня получилось заставить pip дополнять команды и опции, но не названия пакетов, к сожалению.

Судя по найденной информации, для работы с автодополнением имен пакетов требуется дополнительно установить **pip-cache** - локальный индекс PyPI. Устанавливать лишнее избыточное ПО в систему я не люблю. Однако, если вам это интересно, можете попробовать:
<font color="#00aa00"><pre>$ pip install pip-cache
$ pip-cache update
</pre></font>
Автодополнение - хоть и удобно, однако, нещадно тормозит. Автодополнение команд:
<font color="#00aa00"><pre>$ pip [TAB]
bundle   freeze   help    install    search   uninstall  unzip   zip
</pre></font>
<font color="#00aa00"><pre>$ pip in[TAB]
$ pip install
</pre></font>
Пример автодополнения опций:
<font color="#00aa00"><pre>$ pip install --[TAB][TAB]
--allow-all-external        --global-option=            --pre
--allow-external=           --help                      --process-dependency-links
--allow-unverified=         --ignore-installed          --proxy=
--build=                    --index-url=                --quiet
--cert=                     --install-option=           --requirement=
--compile                   --log=                      --root=
--download=                 --log-file=                 --src=
--download-cache=           --no-clean                  --target=
--editable=                 --no-compile                --timeout=
--egg                       --no-deps                   --upgrade
--exists-action=            --no-download               --user
--extra-index-url=          --no-index                  --verbose
--find-links=               --no-install                --version
--force-reinstall           --no-use-wheel
</pre></font>