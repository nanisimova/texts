# Начало работы с SQLAlchemy ORM

## Используемая терминология

ORM - Object Relational Mapper.

### Mapping и mapper

**Mapping** - это определение соответствия между разными объектами.

**Mapper** - это <b>посредник</b>, занимается определением соответствия между разными объектами, преобразованием
данных и их передачей.

Подобный прием в программировании встречается настолько часто, что "Data Mapper" выделен как отдельный шаблон проектирования.

### Шаблон проектирования Data Mapper

**Data Mapper** - как частный случай паттерна Mapper.

Пример, как выполняется обновление записи в базе данных, если в разработке использован шаблон "Data Mapper":
```python
client = Client()
client.name = 'Dionis'
mapper = Mapper(db_connect)
mapper.save(client)
```
Существует две разных сущности - объект с данными и объект-mapper, который отвечает за обновление данных
в соответствующих таблицах БД. В отличие от **ActiveRecord**, **Data Mapper** предполагает разделение данных и логики
взаимодействия с БД.

### Шаблон проектирования ActiveRecord

Шаблон **ActiveRecord** имеет много общего с шаблоном **Data Mapper**. Оба паттерна часто используются для преобразования
данных из реляционного представления в объектное.

**ActiveRecord** - это принцип, при котором каждая таблица базы данных обернута в класс, а объект привязан к строке
в таблице БД. Изменяются данные объекта -&gt; изменяется содержимое строки. Логика взаимодействия с БД
находится в самом объекте, за собственное сохранение объект отвечает сам.

Пример, как происходит обновление данных, если в разработке использован шаблон **ActiveRecord**:
```python
client = Client(db_connect)
client.name = 'Dionis'
client.save()
```
Большинство ORM реализуют паттерн **ActiveRecord**. Так, например, **Django ORM** реализует паттерн **ActiveRecord**,
тогда как **SQLAlchemy** – **Data Mapper**.

## Примеры использования SQLAlchemy ORM

SQLAlchemy ORM сопоставляет определяемые пользователем классы Python с таблицами базы данных, а экземпляры этих классов (объекты) - со строками в соответствующих таблицах.

ORM включает в себя:
<ul>
<li><b>систему синхронизации всех изменений</b> в состояниях между объектами и связанными с ними строками,</li>
<li><b>систему реализации запросов к базе данных</b> в терминах объектной модели, без использования SQL.</li>
</ul>
SQLAlchemy ORM в своей основе использует <a href="sqlalchemy-expression-language.md">SQLAlchemy Expression Language</a>.

**Преимущества ORM**

<ul>
<li>Комфорт. Формирование select-запросов в SQLAlchemy ORM не намного проще составления аналогичного SQL-запроса, но обновление, удаление или добавление новых данных - просто удовольствие.</li>
<li>Переносимость. SQLAlchemy позволяет писать код, который будет работать с разными базами данных.</li>
<li>Безопасность. Параметры запросов экранируются.</li>
</ul>

**Недостатки ORM**

<ul>
<li>Потеря в гибкости. Написать сложный запрос, со множеством условий и десятком джойнов проще вручную, чем с помощью даже самой совершенной ORM. Код сложного запроса часто будет читабельнее в виде SQL, чем в формате очень длинной записи SQLAlchemy.</li>
<li>Потеря в скорости. ORM всегда будет медленней, чем чистый SQL-запрос. Ведь по сути, ORM - это жирная и сложная прослойка, которая не является необходимой.</li>
</ul>

### Создаем классы

Чтоб начать работу с БД посредством ORM, первое, что нужно сделать - это описать структуру таблиц базы
данных в формате объектной модели. Здесь и далее я привожу код скриптов в полном объеме. Это плохо сказывается
на продвижении, зато - намного нагляднее, чем отдельные куски кода.
```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Text, create_engine, select
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class Client(Base):
    __tablename__ = 'clients'

    id = Column(Integer, primary_key=True)
    name = Column(Text)
    phone = Column(String(30))

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)

Session = sessionmaker(bind=engine)
session = Session()
stmt = select([Client]).where(Client.id == 1)
result = session.execute(stmt)
print(result.fetchall())
```

После создания необходимых классов, создаем новую сессию и выполняем запрос для проверки работоспособности того,
что мы понаписали, например, запрос на получение данных из БД.

Для подключения к БД и выполнения запроса используем класс Session. Сессия - это механизм управления процессом
взаимодействия с базой данных. В рамках сессии устанавливается соединение, сессия сохраняет состояние объектов
таблиц и выполняет коммиты по требованию, посредством сессии выполняются запросы.

Вывод:
<pre>
$ python3 select_data_1.py
2021-01-27 INFO sqlalchemy.engine.base.Engine SELECT clients.id, clients.name,
clients.phone
FROM clients
WHERE clients.id = %(id_1)s
2021-01-27 INFO sqlalchemy.engine.base.Engine {'id_1': 1}
[(1, 'Vasiliy', '+794567899876')]
</pre>

### Динамическое создание класса таблицы на основе информации, полученной из БД

Данный код является отличным примером для демонстрации работы mapper.
```python
from sqlalchemy import MetaData, Table, create_engine, select
from sqlalchemy.orm import mapper, sessionmaker

engine = create_engine('mysql+pymysql://root:12345@localhost/test', echo=True)
metadata = MetaData()

client_table = Table("clients", metadata, autoload_with=engine, extend_existing=True)

class Client(object):
    pass

mapper(Client, client_table)

Session = sessionmaker(bind=engine)
session = Session()
print(session.execute(select([Client])).fetchall())
```

Запускаем скрипт:
<pre>$ python3 select_data_5.py
...
2021-01-28 INFO sqlalchemy.engine.base.Engine SHOW CREATE TABLE `clients`
2021-01-28 INFO sqlalchemy.engine.base.Engine SELECT clients.id, clients.name, clients.phone
FROM clients
2021-01-28 INFO sqlalchemy.engine.base.Engine {}
[(1, 'Vasiliy', '+794567899876'),
(2, 'Igor', '+794567899876'),
(3, 'Sergey', '+794567899876'),
(5, 'Doris', '+791698778654'),
(6, 'Anna', '+791566678478'),
(7, 'Svetlana', '+791693712121'),
(10, 'Denis', '+79469008767553')]
root@hp3:/home/root/alch#
</pre>
Среди всех выполняемых SQLAlchemy запросов, видим "SHOW CREATE TABLE", который и становится основой для автоматического
наполнения класса Client содержимым.

### Как добавить данные в таблицу

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Text, create_engine
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class Client(Base):
    __tablename__ = 'clients'

    id = Column(Integer, primary_key=True)
    name = Column(Text)
    phone = Column(String(30))

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
Session = sessionmaker(bind=engine)
session = Session()
session.add( Client(name='Denis', phone='+79469008767553') )
session.commit()
```

Несмотря на мое очень скептичное отношение к ORM в целом, очевидно, что добавить данные в таблицу стало значительно
проще, чем если бы вручную писали инструкцию 'INSERT INTO...'.

Метод сессии **add()** только готовит данные к добавлению. Окончательная отправка в базу данных произойдет только
после выполнения **commit()**. До тех пор, пока **commit()** не выполнен, можно применить **session.rollback()** и сбросить
список запланированных изменений.

### Обновление данных в таблице

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Text, create_engine
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class Client(Base):
    __tablename__ = 'clients'

    id = Column(Integer, primary_key=True)
    name = Column(Text)
    phone = Column(String(30))

    def __repr__(self):
        return "&lt;Client(name='%s', phone='%s')&gt;" % (self.name, self.phone)

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
Session = sessionmaker(bind=engine)
session = Session()

client = session.query(Client).filter_by(id=9).first()
client.name = 'Dionis'
session.commit()
```

До тех пор, пока вы не вызовете метод **commit()** - никакие изменения вноситься в БД не будут.

**session.flush()** - передает серию операций в базу данных (вставка, обновление, удаление). **session.commit()**
фиксирует эти изменения в базе данных и завершает текущую транзакцию. Метод **session.commit()** автоматически
вызывает **session.flush()**, но при необходимости **session.flush()** может вызываться программистом в виде
независимой операции. Используя **flush()** следует учитывать, поддерживает БД транзакции или нет.

Результат выполнения скрипта на изменение данных:
<pre>
$ python3 update_data_1.py
2021-01-27 INFO sqlalchemy.engine.base.Engine {'id_1': 9, 'param_1': 1}
2021-01-27 INFO sqlalchemy.engine.base.Engine UPDATE clients SET name=%(name)s
WHERE clients.id = %(clients_id)s
2021-01-27 INFO sqlalchemy.engine.base.Engine {'name': 'Dionis', 'clients_id': 9}
2021-01-27 INFO sqlalchemy.engine.base.Engine COMMIT
</pre>

### Удаление данных из таблицы

Для удаления строк из таблицы базы данных используем метод session.delete():
```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Text, create_engine
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class Client(Base):
    __tablename__ = 'clients'

    id = Column(Integer, primary_key=True)
    name = Column(Text)
    phone = Column(String(30))

    def __repr__(self):
        return "&lt;Client(name='%s', phone='%s')&gt;" % (self.name, self.phone)

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
Session = sessionmaker(bind=engine)
session = Session()

client = session.query(Client).filter_by(id=9).first()
session.delete(client)
session.commit()
```

Вывод:
<pre>$ python3 delete_data_1.py
2021-01-27 INFO sqlalchemy.engine.base.Engine DELETE FROM clients WHERE clients.id = %(id)s
2021-01-27 INFO sqlalchemy.engine.base.Engine {'id': 9}
2021-01-27 INFO sqlalchemy.engine.base.Engine COMMIT
</pre>

### Запрос данных из двух связанных таблиц. Определение связи между двумя таблицами

До этого момента мы работали только с одной таблицей, теперь добавим вторую и определим между ними отношения
(связь один-ко-многим):
```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, ForeignKey, Integer, String, Text, create_engine
from sqlalchemy.orm import relationship, sessionmaker

Base = declarative_base()

class Client(Base):
    __tablename__ = 'clients'

    id = Column(Integer, primary_key=True)
    name = Column(Text)
    phone = Column(String(30))

    def __repr__(self):
        return "&lt;Client(name='%s', phone='%s')&gt;" % (self.name, self.phone)

class Message(Base):
    __tablename__ = 'messages'

    id = Column(Integer, primary_key=True)
    client_id = Column(Integer, ForeignKey('clients.id'))
    text = Column(String(255))

    client = relationship("Client", back_populates="messages")

    def __repr__(self):
        return "&lt;Message(client_id='%d', text='%s')&gt;" % (self.client_id, self.text)

Client.messages = relationship("Message", order_by=Message.id, back_populates="client")

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
Session = sessionmaker(bind=engine)
session = Session()
client = session.query(Client).filter_by(id=1).first()
print(client)
print(client.messages)
```

После описания таблиц формируем запрос, мы хотим получить данные о клиенте под номером 1 и список сообщений,
которые были отправлены клиенту. При запуске скрипта получаем отладочную информацию и тщательно ее изучаем:
<pre>
$ python3 select_data_3.py
2021-01-27 INFO sqlalchemy.engine.base.Engine SELECT clients.id AS clients_id,
clients.name AS clients_name, clients.phone AS clients_phone
FROM clients
WHERE clients.id = %(id_1)s
 LIMIT %(param_1)s

2021-01-27 INFO sqlalchemy.engine.base.Engine {'id_1': 1, 'param_1': 1}
&lt;Client(name='Vasiliy', phone='+794567899876')&gt;

2021-01-27 INFO sqlalchemy.engine.base.Engine SELECT messages.id AS messages_id,
messages.client_id AS messages_client_id, messages.`text` AS messages_text
FROM messages
WHERE %(param_1)s = messages.client_id ORDER BY messages.id
2021-01-27 INFO sqlalchemy.engine.base.Engine {'param_1': 1}
[&lt;Message(client_id='1', text='Бесплатные звонки заканчиваются сегодня')&gt;,
&lt;Message(client_id='1', text='Кредит всего под 65% годовых!')&gt;]
</pre>
Как мы видим, ORM отправила в базу данных два запроса, первый SELECT получил информацию из таблицы clients,
второй - из messages.

Это не совсем тот результат, на который я рассчитывала. Попробуем обойтись одним запросом, вместо двух:
```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, ForeignKey, Integer, String, Text, create_engine, select
from sqlalchemy.orm import relationship, sessionmaker

Base = declarative_base()

class Client(Base):
    __tablename__ = 'clients'

    id = Column(Integer, primary_key=True)
    name = Column(Text)
    phone = Column(String(30))

    def __repr__(self):
        return "&lt;Client(name='%s', phone='%s')&gt;" % (self.name, self.phone)

class Message(Base):
    __tablename__ = 'messages'

    id = Column(Integer, primary_key=True)
    client_id = Column(Integer, ForeignKey('clients.id'))
    text = Column(String(255))

    client = relationship("Client", back_populates="messages")

    def __repr__(self):
        return "&lt;Message(client_id='%d', text='%s')&gt;" % (self.client_id, self.text)

Client.messages = relationship("Message", order_by=Message.id, back_populates="client")

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
Session = sessionmaker(bind=engine)
session = Session()

clients = session.query(Client, Message).filter(Client.id == Message.client_id,) \
    .filter(Client.id == 1,).all()
print(clients)
```

Вывод:
<pre>$ python3 select_data_3.py
2021-01-27 INFO sqlalchemy.engine.base.Engine SELECT clients.id AS clients_id,
clients.name AS clients_name, clients.phone AS clients_phone,
messages.id AS messages_id, messages.client_id AS messages_client_id,
messages.`text` AS messages_text
FROM clients, messages
WHERE clients.id = messages.client_id AND clients.id = %(id_1)s

2021-01-27 INFO sqlalchemy.engine.base.Engine {'id_1': 1}
[(&lt;Client(name='Vasiliy', phone='+794567899876')&gt;,
&lt;Message(client_id='1', text='Бесплатные звонки заканчиваются сегодня')&gt;),
(&lt;Client(name='Vasiliy', phone='+794567899876')&gt;,
&lt;Message(client_id='1', text='Кредит всего под 65% годовых!')&gt;)]
</pre>
Ситуация улучшилась. Теперь это один запрос.

### JOIN-запросы

Для составления JOIN-запроса используем метод **join()**:
```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, ForeignKey, Integer, String, Text, create_engine, select
from sqlalchemy.orm import relationship, sessionmaker

Base = declarative_base()

class Client(Base):
    __tablename__ = 'clients'

    id = Column(Integer, primary_key=True)
    name = Column(Text)
    phone = Column(String(30))

    def __repr__(self):
        return "&lt;Client(name='%s', phone='%s')&gt;" % (self.name, self.phone)

class Message(Base):
    __tablename__ = 'messages'

    id = Column(Integer, primary_key=True)
    client_id = Column(Integer, ForeignKey('clients.id'))
    text = Column(String(255))

    client = relationship("Client", back_populates="messages")

    def __repr__(self):
        return "&lt;Message(client_id='%d', text='%s')&gt;" % (self.client_id, self.text)

Client.messages = relationship("Message", order_by=Message.id, back_populates="client")

engine = create_engine('mysql+pymysql://root:mypassword@localhost/test', echo=True)
Session = sessionmaker(bind=engine)
session = Session()

clients = session.query(Client, Message).join(Message).all()
print(clients)
```

Результат, список всех клиентов, которые получали хоть какие-то сообщения:
<pre>$ python3 select_data_4.py
2021-01-27 INFO sqlalchemy.engine.base.Engine SELECT clients.id AS clients_id,
clients.name AS clients_name, clients.phone AS clients_phone,
messages.id AS messages_id, messages.client_id AS messages_client_id,
messages.`text` AS messages_text
FROM clients INNER JOIN messages ON clients.id = messages.client_id

2021-01-27 INFO sqlalchemy.engine.base.Engine {}
[(&lt;Client(name='Vasiliy', phone='+794567899876')&gt;,
&lt;Message(client_id='1', text='Бесплатные звонки заканчиваются сегодня')&gt;),
(&lt;Client(name='Vasiliy', phone='+794567899876')&gt;,
&lt;Message(client_id='1', text='Кредит всего под 65% годовых!')&gt;),
(&lt;Client(name='Igor', phone='+794567899876')&gt;,
&lt;Message(client_id='2', text='Кредит всего под 65% годовых!')&gt;)]
</pre>
Если в приведенном коде мы заменим всего одно слово, то вместо запроса INNER JOIN получим запрос LEFT OUTER JOIN и совсем другой результат. Изменяем join() на outerjoin():

```python
clients = session.query(Client, Message).outerjoin(Message).order_by(Client.name).all()
```

Получаем вывод, список всех клиентов и отправленные им сообщения (если таковые есть):
<pre>$ python3 select_data_4.py
2021-01-27 INFO sqlalchemy.engine.base.Engine SELECT clients.id AS clients_id,
clients.name AS clients_name, clients.phone AS clients_phone,
messages.id AS messages_id, messages.client_id AS messages_client_id,
messages.`text` AS messages_text
FROM clients
LEFT OUTER JOIN messages ON clients.id = messages.client_id
ORDER BY clients.name

2021-01-27 INFO sqlalchemy.engine.base.Engine {}
[(&lt;Client(name='Anna', phone='+791566678478')&gt;, None),
(&lt;Client(name='Doris', phone='+791698778654')&gt;, None),
(&lt;Client(name='Igor', phone='+794567899876')&gt;,
&lt;Message(client_id='2', text='Кредит всего под 65% годовых!')&gt;),
(&lt;Client(name='Sergey', phone='+794567899876')&gt;, None),
(&lt;Client(name='Svetlana', phone='+791693712121')&gt;, None),
(&lt;Client(name='Vasiliy', phone='+794567899876')&gt;,
&lt;Message(client_id='1', text='Бесплатные звонки заканчиваются сегодня')&gt;),
(&lt;Client(name='Vasiliy', phone='+794567899876')&gt;,
&lt;Message(client_id='1', text='Кредит всего под 65% годовых!')&gt;)]
</pre>
Работать с JOIN-ами в SQLAlchemy значительно проще, чем изначально можно было ожидать.

### Полезные ссылки
<a href="https://docs.sqlalchemy.org/en/14/orm/tutorial.html" rel="nofollow">sqlalchemy.org: Object Relational Tutorial (1.x API)</a>


