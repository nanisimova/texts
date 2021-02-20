# Что такое HTML::FormFu в perl

<i>Что такое HTML::FormFu в perl. Как использовать HTML::FormFu, простые примеры. Создание форм, обработка данных. Введение в HTML::FormFu для начинающих.</i>

&nbsp;
<h2>Что такое HTML::FormFu и для каких задач его можно использовать</h2>
<font color="#00aa00">HTML::FormFu</font> - это фреймворк для работы с формами. Предоставляет инструментарий для создания и проверки форм, загрузки и обновления данных, включая сохранение полученных данных в БД.

Благодаря дополнительным плагинам, HTML::FormFu может использоваться совместно с <font color="#00aa00">Catalyst</font> и <font color="#00aa00">DBIx::Class</font>.

HTML::FormFu считается одной из самых функциональных систем для работы с формами в Perl. Кроме него, популярны <font color="#00aa00">Form::Processor</font>, <font color="#00aa00">CGI::FormBuilder</font>, <font color="#00aa00">HTML::FormFu</font>, <font color="#00aa00">Rose::HTML::Form</font>.

CGI::FormBuilder имеет схожую с HTML::FormFu функциональность, но HTML::FormFu считается более гибким и мощным.

*На мой взгляд, HTML::FormFu, действительно, потрясающе мощный инструмент, основным недостатком которого является отсутствие нормальных руководств по использованию. Очень мало примеров применения. Даже в англоязычном сегменте интернета, про русскоязычный даже говорить не стоит. Из-за этого, первое знакомство с HTML::FormFu вызывает смешанные чувства: "Зачем сюда запихали этого мамонта, неужели нельзя было сделать проще?"*

## Возможности HTML::FormFu

**1. Создание форм**

HTML::FormFu генерирует html-код формы на основе конфигурационных данных.

**2. Заполнение форм**

HTML::FormFu может добавить данные в созданную форму. Например, значения по-умолчанию, или повторное заполнение формы данными, в случае, возникновения ошибок при обработке полей.

**3. Проверка введенных в форму данных**

HTML::FormFu может проверить введенные пользователем данные на соответствие заданным условиям.

**4. Вывод ошибок**

С помощью HTML::FormFu можно проверить введенные пользователем данные, и если обнаружены ошибки - вывести сообщение, и выделить поля с некорректными данными.

**5. Сохранение данных**

HTML::FormFu может помочь вам сохранить введенные данные в БД.

## Простые примеры использования HTML::FormFu

Для того, чтобы можно было использовать HTML::FormFu, желательно установить модули <font color="#00aa00">CGI</font> и <font color="#00aa00">Template</font>.

### Создание и вывод формы с помощью HTML::FormFu. Пример 1

Для реализации простейшей формы на HTML::FormFu, создаем три файла: yml-конфиг, perl-скрипт и шаблон template toolkit.

Приведенный ниже пример - это только создание формы. Без обработки и сохранения введенных пользователем данных.

#### Perl-скрипт form.pl

```perl
#!/usr/bin/perl

use HTML::FormFu;
use CGI;
use Template;

my $config = {
	INTERPOLATE  => 1,
	POST_CHOMP   => 1,
};

my $q = CGI->new;
my $form = HTML::FormFu->new;
my $template = Template->new($config);

print "Content-type: text/html\n\n";

# form.yml содержит формализованное описание полей формы и дополнительные параметры
$form->load_config_file('form.yml');
# $form - содержит полностью готовый html-код формы, требуется 
# только отобразить его в шаблоне
$template->process('template.html', { form => $form } ); 
```

#### Конфиг form.yml

```yaml

---
action: /login
indicator: submit
auto_fieldset: 1

elements:
      - type: Text
        name: user
        constraints:
          - Required

      - type: Password
        name: pass
        constraints:
          - Required

      - type: Submit
        name: submit

constraints:
      - SingleValue
```

#### Шаблон template.html

```html
<meta http-equiv="content-type" content="text/html; charset=utf-8">
<title>Форма</title>
[% form %]
```

После того, как все файлы созданы, запускаем web-сервер (я использовала Apache) и вводим в адресной строке браузера: <font color="#00aa00">http://localhost:8080/cgi-bin/form.pl</font>.

Будет выведено два поля для ввода логина и пароля, и кнопка "Отправить запрос". Все это будет аккуратно обведено тонкой рамочкой.

### Создание и вывод формы с помощью HTML::FormFu. Пример 2

Второй пример использования HTML::FormFu в perl рассматривает создание и вывод формы на html-странице, без создания yml-конфига. Параметры формы задаются непосредственно в скрипте.

Создаем два файла: perl-скрипт и шаблон template toolkit.

#### Perl-скрипт form.pl

```perl
#!/usr/bin/perl

use HTML::FormFu;
use CGI;
use Template;

my $config = {
	INTERPOLATE  => 1,
	POST_CHOMP   => 1,
};

my $q = CGI->new;
my $form = HTML::FormFu->new;
my $template = Template->new($config);

print "Content-type: text/html\n\n";

$form->action("/login");
$form->indicator("submit");
$form->auto_fieldset(1);
$form->elements([
    {
        name        => 'user',
        type        => 'Text',
        constraints => ['Required'],
    },
    {
        name        => 'pass',
        type        => 'Password',
        constraints => ['Required'],
    },
    {
        name => 'submit',
        type => 'Submit'
    }
]);

$form->constraints(['SingleValue']);
my $vars = {
        form => $form,
};
$template->process('template.html', $vars);
```

#### Шаблон template.html

```html
<meta http-equiv="content-type" content="text/html; charset=utf-8">
<title>Форма</title>
[% form %]
```

### Создание, вывод формы, обработка введенных данных с помощью HTML::FormFu. Пример 3

Третий вариант создания формы. Точно так же, как и во втором варианте, конфигурационные данные формы содержатся в скрипте. Но задаются не с помощью отдельных методов, а сразу, в общем большом хеше.

Кроме того, в скрипт добавлены элементы обработки формы. Получаем данные, проверяем их, возвращаем ошибку, если юзер не подходящий.

#### Perl-скрипт form.pl

```perl
#!/usr/bin/perl

use HTML::FormFu;
use CGI;
use Template;

my $config = {
	INTERPOLATE  => 1,
	POST_CHOMP   => 1
};

my $q = CGI->new;
my $form = HTML::FormFu->new;
my $template = Template->new($config);
print "Content-type: text/html\n\n";

my $definition = {
    action => "form.pl",
    indicator => "submit",
    auto_fieldset => 1,

    elements => [ {
        name        => 'user',
        type        => 'Text',
        constraints => {
            type => 'Callback',
            message => 'Text of user error'
        },
    }, {
        name        => 'pass',
        type        => 'Password',
    }, {
        name => 'submit',
        type => 'Submit'
    } ],
};
$form->populate($definition);
$form->process( $q );

if ( $form->submitted_and_valid ) {
    # авторизацию может пройти только юзер с именем 'User1',
    # остальным возвращаем ошибку
    unless ($form->param('user') eq 'User1') {
        $form->get_field('user')
            ->get_constraint({ type => 'Callback' })
            ->force_errors(1);

        $form->process;
    }
}
$template->process('template.html', { form => $form });
```

Небольшой комментарий. Несмотря на такой изысканный способ получения введенных в форму данных, через методы объекта HTML::FormFu, данные можно получить с помощью обычного CGI. Иногда это может оказаться полезным.

```perl
my $q = CGI->new;
my $params = $q->Vars;

my $vals;
$vals->{'username'} = $params->('user');
$vals->{'password'} = $params->{'pass'};
```

#### Шаблон template.html

```html
<meta http-equiv="content-type" content="text/html; charset=utf-8">
<title>Форма</title>
[% form %]
```

