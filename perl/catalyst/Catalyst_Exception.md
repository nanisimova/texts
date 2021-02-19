# Catalyst::Exception

Catalyst::Exception - модуль для работы с исключениями.

Catalyst::Exception наследует методы Catalyst::Exception::Base, если вы не захотели задать какой-то иной класс для работы с исключительными ситуациями в Catalyst-приложении.

Обычно в модулях для работы с исключениями есть метод throw - для генерации исключений (как говорят, "выбросить"). Код, в котором происходит вызов исключения, принято оформлять с помощью try. Для того, чтобы исключение перехватить, обычно используется метод catch ("поймать").

Catalyst::Exception предоставляет для работы метод throw().

## Методы

**throw**

```perl
throw( $message )
throw( message => $message )
throw( error => $error )
```

Метод throw() генерирует исключение. Выполнение запроса будет экстренно завершено и пользователю возвращено сообщение об ошибке.

На самом деле, метод throw() является оберткой для функции croak() модуля Carp.
<blockquote>Модуль Carp предоставляет альтернативу стандартным функциям perl - warn() и die(). Функция carp() вместо warn(), функции croak() (для коротких сообщений) и confess() (для длинных сообщений) вместо die().

Смысл использования croak() вместо die() - разный вывод сообщений об ошибках.

Но по сути, это все те же warn() и die(), просто с предварительно отформатированным выводом.</blockquote>
Если вы вызовете в вашем catalyst-приложении die() :

```perl
sub order_list : Local Args(0) Path('/user/order_list') {
    die "Unknown error";
}
```

то получите вот такой вывод:

<img src="catalyst_exception_croak.png" alt="catalyst_exception_croak" class="aligncenter size-full wp-image-1816" width="597" height="212">

Пользователь увидит на экране имя вашего проблемного файла и номер строки с ошибкой.

Если вы будете использовать Catalyst::Exception-&gt;throw() :

```perl
sub order_list : Local Args(0) Path('/user/order_list') {

    Catalyst::Exception->throw("Unknown error");
}
```

то внешний вид выводимой ошибки изменится:

<img src="catalyst_exception_message.png" alt="catalyst_exception_message" class="aligncenter size-full wp-image-1817" width="597" height="239">
