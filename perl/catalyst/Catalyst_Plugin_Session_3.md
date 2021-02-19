# Создание системы сессий, авторизации и аутентификации в Catalyst. Часть 3. Роли пользователей

*Использование ролей в Catalyst. Плагин Catalyst::Plugin::Authorization::Roles. Создание таблиц с ролями в БД, установка ограничений доступа в контроллере.*

<a href="Catalyst_Plugin_Session_1.md">Создание системы сессий, авторизации и аутентификации в Catalyst. Часть 1. Сессии в Catalyst</a>

<a href="Catalyst_Plugin_Session_2.md">Создание системы сессий, авторизации и аутентификации в Catalyst. Часть 2. Аутентификация и авторизация</a>

Роли в Catalyst используются для создания пользователей с различными правами доступа. Например, пользователь с правами администратора будет иметь доступ ко всем управляющим элементам сайта, а пользователь manager - только к разделу по генерации отчетности.

После работы, выполненной на предыдущем шаге, добавить использование ролей будет совсем просто.

**1.** Создаем в БД таблицу ролей и таблицу для установления связи между пользователем и назначенной ролью.

Каждому пользователю может быть назначено несколько ролей. Т.е. пользователь может одновременно получить роли "Менеджера" и "Бухгалтера" - и иметь доступ к соответствующим разделам. Каждая новая роль расширяет полномочия одного клиента.

<pre>CREATE TABLE roles (id INT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY, name CHAR(30));

CREATE TABLE user_roles (user_id INT(11) NOT NULL, role_id INT(11) NOT NULL);
ALTER TABLE user_roles
    ADD CONSTRAINT user_roles_user_id_fkey FOREIGN KEY (user_id)
    REFERENCES users(id) ON DELETE CASCADE ON UPDATE CASCADE;

ALTER TABLE user_roles
    ADD CONSTRAINT user_roles_role_id_fkey FOREIGN KEY (role_id)
    REFERENCES roles(id) ON DELETE CASCADE ON UPDATE CASCADE;

CREATE INDEX fki_user_roles_id_fkey
  ON user_roles (user_id, role_id);
</pre>

**2.** Создаем несколько ролей и устанавливаем связь между ролью и пользователем.

<pre>
INSERT INTO roles (name) VALUES ("admin");
INSERT INTO roles (name) VALUES ("manager");

INSERT INTO user_roles (user_id, role_id) VALUES (5, 1);
INSERT INTO user_roles (user_id, role_id) VALUES (4, 2);
</pre>

**3.** Устанавливаем дополнительный плагин для Catalyst.

<pre>cpan force install Catalyst::Plugin::Authorization::Roles</pre>

**4.** Создаем новый контроллер для демонстрации работы с ролями.

<pre>perl script/app_create.pl controller Admin</pre>

**5.** Указываем новый плагин Authorization::Roles в файле lib/app.pm:

```perl
use Catalyst qw/
    Authorization::Roles
    Authentication
    Session
    Session::PerUser
    Session::Store::DBI
    Session::State::Cookie
/;
```

**6.** Определяем права доступа к некому участку кода.

*lib/app/Controller/Admin.pm*:

```perl
sub index :Path :Args(0) {
    my ( $self, $c ) = @_;

    unless ($c->check_user_roles('admin')) {
        $c->response->body('Matched app::Controller::Admin in Admin. Access denied');
    } else {
        $c->response->body('Matched app::Controller::Admin in Admin. Welcome!');
    }
}
```

### Методы Catalyst::Plugin::Authorization::Roles

**assert_user_roles [ $user ], @roles**

Проверка пользователя на принадлежность к некому списку ролей. Если проверка не удалась (пользователь $c-&gt;user не определен, пользователю не назначены указанные роли), будет возвращена ошибка. Организовать перехват ошибки можно с помощью eval. Если первый аргумент не указан, по умолчанию будет использовано значение $c-&gt;user.

**check_user_roles [ $user ], @roles**

Принимает те же аргументы, что и assert_user_roles, выполняет те же проверки, но вместо ошибки возвращает true или false.

**assert_any_user_role [ $user ], @roles**

Проверяет принадлежность пользователя хотя бы к одной из указанных ролей. В остальном, поведение аналогично assert_user_roles.

**check_any_user_role [ $user ], @roles**

Возвращает true или false, в зависимости от результатов проверки - задана ли для пользователя хотя бы одна из указанных ролей.

**7.** Проверяем работоспособность кода.

Запускаем сервер, пробуем зайти на страницу *http://localhost:3000/admin/* и получить сообщение "Welcome!". Это получится только после авторизации под пользователем admin.

