# Catalyst и его Chained

*Что такое атрибут Chained в Catalyst. Что такое атрибуты Path, CaptureArgs и Args. Использование атрибутов Local, Global, Private.*

Атрибуты методов контроллеров - это способ связать определенный метод-обработчик с конкретным запросом, по определенному адресу. Именно благодаря атрибутам, Catalyst понимает, какой метод будет обрабатывать запрос к странице http://localhost:3000/page/2 . Более того, атрибуты позволяют создать целую цепочку методов, которые будут последовательно обрабатывать запрос.

Ниже будет рассмотрено использование основных атрибутов методов Catalyst. Данная публикация носит практический характер: много примеров, мало текста.

Все приведенные примеры были проверены на основе специально созданного контроллера Chain.pm и шаблона chain.tt . При этом, содержимое методов оставалось практически неизменным (для чистоты экспериментов), изменялись только атрибуты. Именно поэтому, в большинстве примеров содержимое методов не приводится, только их объявления.

*/lib/MyApp/Controller/Chain.pm:*

```perl
package MyApp::Controller::Chain;
use Moose;
use namespace::autoclean;

BEGIN { extends 'Catalyst::Controller'; }

sub index :Path :Args(0) {
    my ( $self, $c ) = @_;

    $c->stash->{template} = 'chain.tt';
    $c->stash->{message}  = 'Its Chain!';

    $c->forward('View::TT');
}

__PACKAGE__->meta->make_immutable;

1;
```

*/root/src/chain.tt:*

<pre>[% message %]</pre>

Вызывается по адресу *http://localhost:3000/chain*


## Args

Атрибут Args позволяет передавать Catalyst-приложению аргументы как составную часть URL. Можно передавать один или несколько аргументов, либо запретить передачу аргументов вообще.

Args(0) - аргументы не разрешены, Args - без указания числа - разрешит неограниченное число аргументов. Args(1) - т.е. Args с указанным числом, разрешит заданное количество аргументов в URL.

Принятые значения аргументов можно извлечь из @_ .

**Пример 1:**
<pre>sub index :Path :Args(0) {}</pre>

Данный обработчик будет привязан к адресу
http://localhost:3000/chain

Попытки вызвать страницу
http://localhost:3000/chain/67
будут завершены ошибкой "Page not found".

**Пример 2:**

```perl
sub index :Path :Args(1) {
    my ( $self, $c, $id ) = @_;

    $c->stash->{template} = 'chain.tt';
    $c->stash->{message}  = $id;

    $c->forward('View::TT');
}
```

Правильный вызов:
http://localhost:3000/chain/67

Не правильный вызов:
http://localhost:3000/chain ,
http://localhost:3000/chain/67/56

**Пример 3:**

```perl
sub index :Path :Args {}
```

Правильный вызов:
http://localhost:3000/chain/67/56 ,
http://localhost:3000/chain/67 ,
http://localhost:3000/chain ,
http://localhost:3000/chain/rr/36

Если передаваемый аргумент не является завершающей частью URL, для указания количества передаваемых аргументов используется атрибут CaptureArgs .

## Path

Атрибут :Path привязывает текущий метод в качестве обработчика к заданному URL.

**Пример 1:**

```perl
sub index :Path('login') :Args(0) {}
```

Правильный вызов:
http://localhost:3000/chain/login

Не правильный вызов:
http://localhost:3000/chain
закончится ошибкой "Page not found"

**Пример 2:**

```perl
sub index :Path('/login') :Args(0) {}
```

Правильный вызов:
http://localhost:3000/login

Не правильный вызов:
http://localhost:3000/chain/login

**Пример 3:**

```perl
sub index :Path('/login/name') :Args(0) {}
```

Правильный вызов:
http://localhost:3000/login/name

Не правильный вызов:
http://localhost:3000/login ,
http://localhost:3000/chain ,
http://localhost:3000/chain/login

## Local
Атрибут :Local - это тоже самое, что :Path('action_name') .

**Пример:**

```perl
sub clear_all :Local :Args(0) {}
```

Правильный вызов:
http://localhost:3000/chain/clear_all

Не правильный вызов:
http://localhost:3000/clear_all ,
http://localhost:3000/chain

## Global
Атрибут Global это аналог атрибута :Path('/action_name').

**Пример:**

```perl
sub clear_all :Global :Args(0) {}
```

Правильный вызов:
http://localhost:3000/clear_all

Не правильный вызов:
http://localhost:3000/chain/clear_all

Считается, что атрибут Global сейчас уже почти не используется, в основном его можно встретить в очень старых Catalyst-приложениях. Вместо Global, для достижения аналогичных результатов используют атрибуты Chained и Path.

## Private

Отличный атрибут, который запрещает доступ к методу. Отныне и вовеки, метод будет доступен только изнутри программы, и никогда - извне.

Атрибут :Private эксклюзивен. Атрибуты :Path или :Chained в его присутствии не действительны. Вызвать метод, в заголовке которого указано :Private, можно с помощью

```perl
$c->forward('clear_all');
```

или

```perl
$c->detach('clear_all');
```

**Пример:**

```perl
sub clear_all :Global :Args(0) :Private {}
```

не смотря на указанные :Global и :Args(0) , обращение по адресу http://localhost:3000/clear_all вернет ошибку - "Страница не найдена"

## PathPart

Атрибут используется при создании цепочек методов для обработки запроса. Привязывает текущий обработчик к некоторой части URL. Если атрибут указан без аргументов, то в качестве значения будет использоваться имя метода.

Записи

```perl
sub action_name :PathPart
```

и
```perl
sub action_name :PathPart('action_name')
```

считаются идентичными по смыслу.

## CaptureArgs

Атрибут CaptureArgs задает количество аргументов, которые передаются как часть URL-строки. Работает так же, как Args, но используется в тех случаях, когда отлавливаемый аргумент не последний элемент в URL-строке.

Например:
**http://localhost:3000/page/{page_id}/rev/{rev_id}**
Для обработки {rev_id} может использоваться Args, а для отлова {page_id} - CaptureArgs.

## Chained

Атрибут Chained позволяет создавать цепочки методов для обработки запросов. Благодаря Chained и PathPart, можно организовать обработку сложных, составных URL, которые содержат множество элементов и атрибутов.

Атрибут Chained содержит название метода, который будет выполнен до того, как выполнится текущий обработчик.

**Пример 1:**

```perl
sub index :Chained('/') :PathPart('user') :CaptureArgs(1) {}
sub edit :Chained('index') :PathPart('edit') :Args(0) {}
```

Правильный вызов:
http://localhost:3000/user/98/edit

Не правильный вызов:
http://localhost:3000/user/98/edit/33 ,
http://localhost:3000/user/98

Обработка запроса с URL "http://localhost:3000/user/98/edit" осуществляется цепочкой методов. Сначала будет вызван метод index, потом, после завершения его работы, вызывается edit.

Данный подход позволяет разнести обработку различных логических объектов по разным методам.

**Пример 2:**

```perl
# После элемента URL "/user" следует один аргумент
sub index :Chained('/') :PathPart('user') :CaptureArgs(1) {}

# Метод обрабатывает URL типа "/user/{user_id}/edit". Аргументы после "/edit" запрещены.
# Сначала вызывается метод index, для обработки части URL "/user/{user_id}", 
# затем отрабатывает текущий метод.
sub edit :Chained('index') :PathPart('edit') :Args(0) {}

# Метод обрабатывает URL типа "/user/{user_id}".
# Сначала вызывается метод index, для обработки части URL "/user/{user_id}", 
# затем отрабатывает текущий метод.
sub view :Chained('index') :PathPart('') :Args(0) {}
```

Правильный вызов:
http://localhost:3000/user/98/edit ,
http://localhost:3000/user/98

Не правильный вызов:
http://localhost:3000/user ,
http://localhost:3000/user/67/edit/promo ,
http://localhost:3000/user/67/view

## Специальные методы в Catalyst

Catalyst совершенно по-особому обрабатывает методы с именами index, auto, begin, end и default. Эти методы вызываются в ключевых точках обработки поступившего запроса.

Стоит быть внимательным, придумывая имена для методов. Я это хорошо запомнила, после того, как для удобства назвала главный обработчик - index, а потом около часа разбиралась, почему метод не работает.


**1.** <font color="#00aa00">default :Path</font> - этот метод вызывается, когда не найдено ни одного подходящего обработчика для запроса. Переопределив его, можно использовать для вывода главной страницы, отображения сообщения об ошибке 404, и т.п.

```perl
package MyApp::Controller::Chain;
use Moose;
use namespace::autoclean;

BEGIN { extends 'Catalyst::Controller'; }

sub main :Chained('/') :PathPart('user') :CaptureArgs(1) {
    my ( $self, $c ) = @_;

    $c->stash->{message} = "User"
}

sub edit :Chained('main') :PathPart('edit') :Args(0) {
    my ( $self, $c ) = @_;

    $c->stash->{template} = 'chain.tt';
    $c->stash->{message}  = $c->stash->{message}.'Its Chain!';
    $c->forward('View::TT');
}

sub default :Path {
    my ( $self, $c ) = @_;

    $c->stash->{template} = 'chain.tt';
    $c->stash->{message}  = $c->stash->{message}.'404';
    $c->forward('View::TT');
}

__PACKAGE__->meta->make_immutable;

1;
```

В этом случае, при обращении к странице http://localhost:3000/chain/ или http://localhost:3000/chain/89/ , вместо стандартной ошибки "Страница не найдена", клиент увидит сообщение об ошибке 404.

**2.** <font color="#00aa00">index :Path :Args(0)</font> - index очень похож на default . Удобен для создания какой-то единой страницы раздела.

```perl
sub index :Path :Args(0) {
    my ( $self, $c ) = @_;

    $c->stash->{template} = 'chain.tt';
    $c->stash->{message}  = $c->stash->{message}.'Hello!!!';
    $c->forward('View::TT');
}
```
Вызывается - http://localhost:3000/chain</li>

**3.** <font color="#00aa00">begin :Private</font> - Вызывается в самом начале обработки запроса, после того, как был определен обрабатывающий контроллер, но до того, как будут вызваны соответствующие URL обработчики. Функция begin будет вызвана в том контроллере, в котором содержится
необходимый метод.
**4.** <font color="#00aa00">end :Private</font> - Вызывается самым последним, после того, как выполнены все методы в цепочке обработки запроса.

Определим в контроллере метод end :

```perl
sub end :Private {
    my ( $self, $c ) = @_;
}
```

Обратимся по адресу http://localhost:3000/chain. В логах сервера можно увидеть, что end был вызван:
<pre>[info] Request took 0.098775s (10.124/s)
.------------------------------------------------------------+-----------.
| Action                                                     | Time      |
+------------------------------------------------------------+-----------+
| /chain/index                                               | 0.094188s |
|  -&gt;MyApp::View::TT-&gt;process                               | 0.093730s |
| /chain/end                                                 | 0.000096s |
'------------------------------------------------------------+-----------'
</pre>

**5.** <font color="#00aa00">auto :Private</font> - Еще один метод. Вызывается после метода begin.

```perl
sub end :Private {
    my ( $self, $c ) = @_;
}

sub begin :Private {
    my ( $self, $c ) = @_;
}

sub auto :Private {
    my ( $self, $c ) = @_;
}
```

<pre>
.------------------------------------------------------------+-----------.
| Action                                                     | Time      |
+------------------------------------------------------------+-----------+
| /chain/begin                                               | 0.000112s |
| /chain/auto                                                | 0.000076s |
| /chain/index                                               | 0.139140s |
|  -&gt; MyApp::View::TT-&gt;process                               | 0.138757s |
| /chain/end                                                 | 0.000096s |
'------------------------------------------------------------+-----------'
</pre>


