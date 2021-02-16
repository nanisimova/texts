# Начало работы SQLAlchemy Expression Language

SQLAlchemy Expression Language (язык выражений) - представляет собой систему представления структур и выражений реляционных баз данных. Позволяет напрямую работать с базами данных.

Expression Language отличается от Object Relational Mapper (ORM), которая является отдельным API, реализованной на основе Expression Language. ORM - высокоуровневая и абстрактная модель, которая сама по себе является примером прикладного использования языка выражений.

Таким образом, SQLAlchemy предоставляет два способа работы с базой данных, либо посредством ORM, либо через более простой и понятный Expression Language. Также возможно сочетание обеих подходов в одном приложении.

## Подготовка к работе с SQLAlchemy

### Установка mariadb на Linux Debian 10 (Buster)

Выполяется под пользователем root:
<pre>
$ apt update
$ apt install mariadb-server
</pre>
После установки проверяем статус нового сервиса:
<pre>$ service mysql status</pre>
Сразу устанавливаем пароль для root и меняем способ аутентификации:
<pre>$ mysql
MariaDB [(none)]&gt;ALTER USER 'root'@'localhost'
IDENTIFIED VIA mysql_native_password;
MariaDB [(none)]&gt;ALTER USER 'root'@'localhost'
IDENTIFIED BY 'mypassword';
MariaDB [(none)]&gt;exit
</pre>
Теперь можно подключаться к БД:
<pre>$ mysql -u root -p</pre>

### Создать базу данных mysql

Чтобы создать базу данных, используем клиента командной строки:
<pre>$ mysql -u username -p
MariaDB [(none)]&gt; show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
MariaDB [(none)]&gt; create database test;
</pre>

### Установка SQLAlchemy

<pre>$ pip install sqlalchemy</pre>

## Работа с SQLAlchemy Expression Language

По сути, вся работа с SQLAlchemy Expression Language сводится к 4 этапам:
<ol>
<li>Импорт необходимых функций и подключение базы данных</li>
<li>Описание структуры базы данных</li>
<li>Составление необходимого sql-выражения</li>
<li>Вызов метода execute и обработка полученных данных</li>
</ol>
При этом, второй и третий этапы могут быть пропущены, см. пример ниже.

### Подключение к базе данных

MariaDB и SQLAlchemy установили, самое время проверить как они будут работать в связке. Для подключения импортируем create_engine(). Первый параметр задается по шаблону:
<pre>dialect+driver://username:password@host:port/database</pre>

*script.py:*
```python
from sqlalchemy import create_engine
engine = create_engine('mysql://root:mypassword@localhost/test', echo=True)
```
Параметр echo=True позволит нам наблюдать за процессом работы SQLAlchemy, которая начнет выводить на экран множество информационных сообщений, в том числе сформированные запросы к БД. Очень полезная возможность и для отладки, и для начального знакомства с SQLAlchemy.

Попытка запуска неожиданно провалилась:
<pre>
$ python3 script.py
ModuleNotFoundError: No module named 'MySQLdb'
</pre>

ПРОСТОЙ способ избавиться от ошибки "No module named MySQLdb" (можно заморочиться с установкой mysqlclient, но после пары попыток, завершившихся ошибкой, мне эта идея перестала нравиться):
<pre>$ pip install pymysql</pre>

Теперь указываем новый драйвер для работы с БД и запускаем скрипт:
```python
engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
```
Если ошибок нет, можно двигаться дальше.

### Показать список таблиц в БД

SQLAlchemy позволяет выполнять чистый sql-код, без предварительного создания классов, структур, без
использования специальных методов. Это может быть очень удобно как на этапе создания каркаса приложения или промежуточного тестирования, так и в тех случаях, когда проще написать запрос руками, чем заставить SQLAlchemy сконструировать его.

Пример:
```python
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
res = engine.execute('SHOW TABLES')
print(res.fetchall())
```

Вывод:
<pre>$ python3 show_tables.py
2021-01-24 18:26:14,530 INFO sqlalchemy.engine.base.Engine
SHOW TABLES
[('clients',), ('messages',)]
</pre>

### Создание таблиц в БД

Создадим две таблицы, связанные друг с другом. Одна будет содержать список клиентов, с именами и номерами
телефонов, вторая - очередь sms-сообщений, которые надо отправить клиентам.

```python
from sqlalchemy import Column, Integer, Text, MetaData, Table, String, ForeignKey, create_engine

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
metadata = MetaData()
clients = Table(
    'clients', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', Text),
    Column('phone', String(30)),
)
messages = Table(
    'messages', metadata,
    Column('id', Integer, primary_key=True),
    Column('client_id', ForeignKey('clients.id')),
    Column('text', String(255)),
)

clients.create(bind=engine, checkfirst=True)
messages.create(bind=engine, checkfirst=True)
```

Заглянем в MariaDB и проверим результат:
<pre>MariaDB [test]&gt; show tables;
+----------------+
| Tables_in_test |
+----------------+
| clients        |
| messages       |
+----------------+
2 rows in set (0.000 sec)
</pre>

### Работа с данными

#### Вставка данных

Добавим нового клиента в таблицу clients:
```python
from sqlalchemy import Column, Integer, MetaData, String, Table, Text, create_engine

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
metadata = MetaData()
clients = Table(
    'clients', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', Text),
    Column('phone', String(30)),
)

rec = clients.insert().values(name='Vasiliy', phone='+794567899876')
engine.execute(rec)
```

Проверим результат:
<pre>MariaDB [test]&gt; select * from clients;
+----+---------+---------------+
| id | name    | phone         |
+----+---------+---------------+
|  1 | Vasiliy | +794567899876 |
+----+---------+---------------+
</pre>
Чтоб добавить в таблицу несколько записей, можно использовать конструкцию:
```python
engine.execute(clients.insert(), [
    {'name' : 'Anna', 'phone' : '+791566678478'},
    {'name' : 'Svetlana', 'phone' : '+791693712121'},
    {'name' : 'Irina', 'phone' : '+749588834114'},
])
```

В таблицу messages поместим сообщение для клиента Василия и выведем на экран id добавленного сообщения:
```python
from sqlalchemy import Column, ForeignKey, Integer, MetaData, String, Table, Text, create_engine

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
metadata = MetaData()

clients = Table(
    'clients', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', Text),
    Column('phone', String(30)),
)
messages = Table(
    'messages', metadata,
    Column('id', Integer, primary_key=True),
    Column('client_id', ForeignKey('clients.id')),
    Column('text', String(255)),
)

rec = messages.insert().values(client_id=1, text='Кредит всего под 65% годовых!')
res = engine.execute(rec)
print(res.inserted_primary_key)
```

Выполняем:
<pre>$ python3 insert_data_2.py
2021-01-24 16:33:40,228 INFO sqlalchemy.engine.base.Engine
INSERT INTO messages (client_id, `text`) VALUES (%(client_id)s, %(text)s)
2021-01-24 16:33:40,229 INFO sqlalchemy.engine.base.Engine
{'client_id': 1, 'text': 'Кредит всего под 65% годовых!'}
2021-01-24 16:33:40,230 INFO sqlalchemy.engine.base.Engine COMMIT
[2]
</pre>

#### Удаление данных из таблицы

Удалим из таблицы всех клиентов под именем "Boris" и выведем на экран количество удаленных строк:
```python
from sqlalchemy import Column, Integer, Text, MetaData, Table, String, \
    ForeignKey, create_engine, select
engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', \
    echo=True)
metadata = MetaData()
clients = Table(
    'clients', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', Text),
    Column('phone', String(30)),
)

rec = clients.delete().where(clients.c.name == 'Boris')
res = engine.execute(rec)
print(res.rowcount)
```

Запускаем скрипт:
<pre>$ python3 delete_data.py
2021-01-24 18:08:52,962 INFO sqlalchemy.engine.base.Engine
DELETE FROM clients WHERE clients.name = %(name_1)s
2021-01-24 18:08:52,962 INFO sqlalchemy.engine.base.Engine
{'name_1': 'Boris'}
2021-01-24 18:08:52,963 INFO sqlalchemy.engine.base.Engine COMMIT
1
</pre>

#### Обновление данных

Переименуем всех клиентов с именем "Boris" и выведем число обработанных строк:
```python
from sqlalchemy import Column, Integer, Text, MetaData, Table, String, ForeignKey, create_engine, select

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
metadata = MetaData()
clients = Table(
    'clients', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', Text),
    Column('phone', String(30)),
)

stmt = clients.update().where(clients.c.name == 'Boris').values(name='Doris')
res = engine.execute(stmt)
print(res.rowcount)
```

Запускаем скрипт:
<pre>$ python3 update_data.py
2021-01-24 18:13:41,321 INFO sqlalchemy.engine.base.Engine
UPDATE clients SET name=%(name)s WHERE clients.name = %(name_1)s
2021-01-24 18:13:41,321 INFO sqlalchemy.engine.base.Engine
{'name': 'Doris', 'name_1': 'Boris'}
2021-01-24 18:13:41,322 INFO sqlalchemy.engine.base.Engine COMMIT
1
</pre>

#### Выборка данных. Пример 1

Получим всех клиентов из таблицы select и выведем на экран одну строку из выборки:
```python
from sqlalchemy import Column, Integer, Text, MetaData, Table, \
    String, ForeignKey, create_engine, select

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', \
    echo=True)
metadata = MetaData()
clients = Table(
    'clients', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', Text),
    Column('phone', String(30)),
)

s_obj = select([clients])
rec = engine.execute(s_obj).fetchone()
print(rec)
```

Вывод:
<pre>$ python3 select_data.py
2021-01-24 16:17:09,620 INFO sqlalchemy.engine.base.Engine
SELECT clients.id, clients.name, clients.phone FROM clients
2021-01-24 16:17:09,621 INFO sqlalchemy.engine.base.Engine {}
(1, 'Vasiliy', '+794567899876')
</pre>

#### Выборка данных. Пример 2

А теперь запросим и выведем на экран все записи из таблицы messages.
```python
from sqlalchemy import Column, Integer, Text, MetaData, Table, String, ForeignKey, create_engine, select

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
metadata = MetaData()
clients = Table(
    'clients', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', Text),
    Column('phone', String(30)),
)
messages = Table(
    'messages', metadata,
    Column('id', Integer, primary_key=True),
    Column('client_id', ForeignKey('clients.id')),
    Column('text', String(255)),
)

s_obj = select([messages])
rec = engine.execute(s_obj).fetchall()
print(rec)
```

Запускаем скрипт:
<pre>$ python3 select_data_2.py
2021-01-24 16:27:55,089 INFO sqlalchemy.engine.base.Engine
SELECT messages.id, messages.client_id, messages.`text` FROM messages
2021-01-24 16:27:55,089 INFO sqlalchemy.engine.base.Engine {}
[(1, 1, 'Бесплатные звонки заканчиваются сегодня')]
</pre>

#### Выборка данных. Пример 3

Еще один пример выборки данных. Проверим, может ли SQLAlchemy Expression Language работать с join-запросами.
Запросим у базы данные обо всех клиентах, которым были отправлены сообщения с рекламой очень выгодного кредита:
```python
from sqlalchemy import Column, Integer, Text, MetaData, Table, String, ForeignKey, create_engine, outerjoin, select

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
metadata = MetaData()
clients = Table(
    'clients', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', Text),
    Column('phone', String(30)),
)
messages = Table(
    'messages', metadata,
    Column('id', Integer, primary_key=True),
    Column('client_id', ForeignKey('clients.id')),
    Column('text', String(255)),
)

j = outerjoin(clients, messages, clients.c.id == messages.c.client_id)
s_obj = select([clients]).select_from(j).where(messages.c.text.like('Кредит%'))
rec = engine.execute(s_obj).fetchall()
print(rec)
```

Запускаем скрипт. Не забываем обратить внимание на выводимые информационные сообщения, среди них мы увидим сформированный SQLAlchemy запрос:
<pre>$ python3 select_data_2.py
2021-01-24 17:43:56,808 INFO sqlalchemy.engine.base.Engine
SELECT clients.id, clients.name, clients.phone
FROM clients LEFT OUTER JOIN messages ON clients.id = messages.client_id
WHERE messages.`text` LIKE %(text_1)s
2021-01-24 17:43:56,809 INFO sqlalchemy.engine.base.Engine
{'text_1': 'Кредит%'}
[(1, 'Vasiliy', '+794567899876'), (2, 'Igor', '+794567899876')]
</pre>

### Полезные ссылки

<a href="https://docs.sqlalchemy.org/en/14/core/tutorial.html" rel="nofollow">sqlalchemy.org: SQL Expression Language Tutorial (1.x API)</a>
