# Начало работы со Sphinx в Django

1. Настраиваем Sphinx, создаем индекс, */etc/sphinxsearch/sphinx.conf*:
<pre>
source src1
{
    type = mysql
    # Параметры подключения к БД
    sql_host = 192.168.102.35
    sql_user = test
    sql_pass = test
    sql_db = dj_aboard
    sql_port = 3306

    sql_query_pre = SET NAMES utf8 COLLATE utf8_unicode_ci
    # запрос который возвращает данные для индексации
    sql_query = SELECT id, id AS 'sid', name, description, user_id FROM catalog_catalog

    sql_attr_uint = sid
    sql_field_string = name
    sql_field_string = description
    sql_attr_uint = user_id
}
index test1
{
    source = src1
    path = /var/lib/sphinxsearch/data/test1
    docinfo = extern
}

indexer
{
    mem_limit = 128M
}

searchd
{   
    listen = 9312 # порт для работы через API
    listen = 9306:mysql41 # порт для комуникаций с MySQL
    log = /var/log/sphinxsearch/searchd.log
    query_log = /var/log/sphinxsearch/query.log
    read_timeout = 5
    max_children = 30
    pid_file = /var/log/sphinxsearch/searchd.pid
    seamless_rotate = 1
    preopen_indexes = 1
    unlink_old = 1
    workers = threads # for RT to work
}
</pre>
Останавливаем демона:
<pre>
/usr/bin/searchd --stop
</pre>

Запускаем индексацию:
<pre>
/usr/bin/indexer --all --verbose
</pre>

Запускаем sphinx:
<pre>
/usr/bin/searchd --config /etc/sphinxsearch/sphinx.conf
</pre>

Проверить, как прошла индексация, можно используя mysql-клиент и стандартные команды SQL:
<pre>
$ mysql -P 9306 -h 0
</pre>

При индексации могут возникнуть проблемы, т.к. sphinx и индексируемая mysql база находятся на разных хостах.

Ошибка **"Can't connect to MySQL server on '192.168.101.35' (111 "Connection refused")"** решается исправлением конфига mysql, в файле /etc/mysql/mariadb.conf.d/50-server.cnf найти строку "bind-address = 127.0.0.1" и закомментировать ее.

Ошибка: **"Host is not allowed to connect to this MariaDB server"**, решается настройкой прав пользователя mysql или созданием нового пользователя со всеми правами:
<pre>
mysql> CREATE USER 'test'@'%' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'test'@'%' WITH GRANT OPTION;
</pre>
вместо
<pre>
mysql> CREATE USER 'test'@'localhost' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'test'@'localhost' WITH GRANT OPTION;
</pre>

2. Создаем view, которое будет обращаться к Sphinx, *core/views.py*:

```python
from sphinxapi import SphinxClient
from django.shortcuts import render
from core.forms import SearchForm

def search(request):
    if request.method == 'POST':
        form = SearchForm(request.POST)
        if form.is_valid():
            search_string = form.cleaned_data['search_string']
            s = SphinxClient()
            s.SetServer('192.168.102.2', 9312)
            s.SetLimits(0, 100)
            if s.Status():
                res = s.Query(search_string)
                return render(request, 'search.html', {'items': res, 'form': form })

    form = SearchForm()
    return render(request, 'search.html', {'form': form })
```

3. Создаем форму для запроса, *core/forms.py*:

```python
from django import forms

class SearchForm(forms.Form):
    search_string = forms.CharField(label = "Find", max_length=255)
```

4. Создаем шаблон для вывода результатов, *templates/search.html*:

```html
<h2>Поиск объекта< /h2>
<form action="/search" method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form }}
    <input type="submit" value="Submit">
</form>
```

*templates/search_form.html:*

```html
{% include "search_form.html" %}

{% for item in items.matches %}
<p>{{ item.attrs.name }}<br>
{{ item.attrs.description }}<br>
<a href="/catalog/view/{{ item.attrs.sid }}">Просмотреть объект</a>
</p>
{% endfor %}
```

5. Новый url добавляем, *core/urls.py*:

```python
from core import views as core_views

urlpatterns = [
    ...
    url(r'^search', core_views.search),
    ...
]
```

