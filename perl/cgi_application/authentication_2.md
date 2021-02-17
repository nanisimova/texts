# CGI::Application::Plugin::Authentication. Работа с зашифрованными паролями

Как уже <a href="authentication.md">говорилось ранее</a>, пароли лучше хранить в зашифрованном виде.

Краткое описание алгоритма:
<ol>
<li>При создании нового пользователя и пароля доступа для него - введенный пароль сохраняем в зашифрованном виде (pass_from_db).</li>
<li>При аутентификации получаем введенный пользователем пароль (pass_from_user) и шифруем его тем же алгоритмом, что и при сохранении пароля.</li>
<li>Получаем из хранилища зашифрованный пароль (pass_from_db) и сравниваем их между собой. Если оба варианта паролей идентичны - все ок.</li>
</ol>


### Работа с зашифрованными паролями при аутентификации

В блоке конфигурации указываем - какой тип шифрования используется для работы с паролями. Введенный пользователем пароль будет зашифрован по указанному алгоритму и после этого, проверен на соответствие хранимому в БД паролю.

```perl
   $self->authen->config(
    DRIVER => [ 'DBI',
      TABLE       => 'user',
      CONSTRAINTS => {
        'user.name'     => '__CREDENTIAL_1__',
        # md5_base64 - указание - какой алгоритм шифрования используется
        'md5_base64:user.password' => '__CREDENTIAL_2__',       
      },
    ],
    STORE	=> 'Session',
    LOGOUT_RUNMODE		=> 'on_logout',
    LOGIN_RUNMODE		=> 'on_login',
    POST_LOGIN_RUNMODE	=> 'on_ok',
    # ниже, соответственно, CREDENTIAL_1 и CREDENTIAL_2
    CREDENTIALS => [ 'authen_username', 'authen_password' ],
  );
```

<font color="#00aa00">CGI::Application::Plugin::Authentication</font> позволяет использовать криптографические алгоритмы - <font color="#00aa00">crypt, MD5 (Digest::MD5), SHA1 (Digest::SHA1)</font>.

Кроме того, можно использовать и собственные фильтры.

### Сохранение пароля в БД в зашифрованном виде

Возможно, реализация шифрования пароля при сохранении его в БД получилась не очень красивой, зато она очень удобна как вариант для ленивых.

```perl
use CGI::Application::Plugin::Authentication::Driver::Filter::md5;

# добавляем нового пользователя
sub add_new_person {
  my $self = shift;
  my $tt_params = {};
  my $q = $self->query();
  
  if ($q->param('username')) {
    my $password = $q->param('password');
    my $username = $q->param('username'); 
	...

    # для шифрования пароля используется та же функция, которая используется
    # внутри CGI::Application::Plugin::Authentication 
    my $filtered = 
      CGI::Application::Plugin::Authentication::Driver::Filter::md5::filter(undef, 
      'base64', $password);
    
    $self->dbh->do(qq{INSERT INTO user SET name=?, password=?}, 
      undef, $username, $filtered);
    $tt_params->{id} = $self->dbh->{'mysql_insertid'};
    ...
  }

  $tt_params->{block} = 'add_new_person';
  return $self->tt_process('template2.tt', $tt_params);
}
```
