# Perl и YAML. Примеры использования модуля Config::YAML

## Что такое YAML?

<font color="#00aa00">YAML</font> — это формат данных, ориентированный на работу со сложными структурами данных. Имеет очень простую, интуитивно понятную систему разметки. В основном, используется как формат для конфигурационных файлов.

Пример конфигурационного файла, в формате yaml:

<pre>
# параметры подключения к базам данных
dev_lab:
  database: 'dev_lab_db'
  host:     'localhost'
  username: 'dev_lab_user'
  password: 'password1'
woman_blog:
  database: 'woman_blog_db'
  host:     'localhost'
  username: 'woman_blog_user'
  password: 'funnypass'
</pre>

И еще один пример:

```yaml
handlers:
   - action_type: send_comment
     handler: comment
   - action_type:
        - get_post
        - send_post
        - remove_post
     handler: post
```

## Примеры использования модуля Config::YAML

<font color="#00aa00">Config::YAML</font> - это объектно-ориентированная обертка вокруг модуля <font color="#00aa00">YAML</font>. Делает работу с конфигурационными файлами очень простой и удобной.

Модуль <font color="#00aa00">Config::YAML</font> позволяет использовать методы <font color="#00aa00">read</font> и <font color="#00aa00">write</font>, для чтения yaml-структуры из файла и ее записи в файл. Метод <font color="#00aa00">read</font> вызывается при создании объекта <font color="#00aa00">Config::YAML</font>.

Кроме того, <font color="#00aa00">Config::YAML</font> предоставляет два метода для работы с элементами структуры данных: <font color="#00aa00">get_[param]</font> и <font color="#00aa00">set_[param]</font>.

<ul>
<li><font color="#00aa00">get_[param]</font> - получает значение заданного элемента из yaml-структуры</li>
<li><font color="#00aa00">set_[param]</font> - задает значение указанному элементу в yaml-структуре</li>
</ul>

### Чтение yaml-файла

Исходное содержимое конфигурационного файла right.yaml:

```yaml
moderator_action:
  - 'edit_comment'
  - 'remove_comment'
  - 'user_block'
  - 'modify_user_profile'
```

**Пример 1**

Пример perl-кода для чтения yaml-файла:

```perl
#!/usr/bin/perl

use strict;
use warnings;

use Config::YAML;
use Data::Dumper;

my $config = Config::YAML->new( config => '../conf/right.yaml' );
    
print Dumper($config);
```

Результат работы скрипта:

<pre>
% perl test_yaml.pl
$VAR1 = bless( {
                 'moderator_action' => [
                                            'edit_comment',
                                            'remove_comment', 
                                            'user_block',
                                            'modify_user_profile'
                                          ],
                 '_strict' => 0,
                 '_infile' => '../conf/right.yaml', ...
</pre>

**Пример 2**

Помимо исходных данных, считанных из yaml файла и преобразованных в сложную структуру данных, модуль <font color="#00aa00">Config::YAML</font> добавит в эту сложную структуру еще несколько записей служебного характера. Подобные записи имеют ключ с префиксом из знака подчеркивания (например, '_infile').

Можно отфильтровать подобные записи с помощью конструкции:

```perl
use Config::YAML;
use Data::Dumper;

my $config = Config::YAML->new( config => '../conf/right.yaml' );
    
my %result = map {
    $_ => $config->{ $_ };
} grep { $_ !~ /^_/ } keys( %{ $config } );
print Dumper(\%result);
```

Результат:
<pre>
% perl test_yaml.pl
$VAR1 = {
          'moderator_action' => [
                                  'edit_comment',
                                  'remove_comment', 
                                  'user_block',
                                  'modify_user_profile'
                                ],
};
</pre>

**Пример 3**

После того, как yaml-файл прочитан, можно обращаться к его элементам, как к обычной сложной структуре данных:

```perl
use Config::YAML;

my $config = Config::YAML->new( config => '../conf/right.yaml' );
print $config->{'moderator_action'}->[1];
```

Вывод скрипта:
<pre>
% perl test_yaml.pl
remove_comment
</pre>

**Пример 4**

Еще один пример обращения к элементам структуры данных:

```perl
my $config = Config::YAML->new( config => '../conf/right.yaml' );

foreach (keys %{$config}) {
	print $_.": ".$config->{$_}."\n";
}
```

Результат:
<pre>
% perl test_yaml.pl
moderator_action: ARRAY(0x5cf6ff0)
_strict: 0
_infile: ../conf/right.yaml
admin_action: ARRAY(0x5cf7000)
_outfile: ../conf/right.yaml
</pre>

### Создание файла yaml из сложной структуры данных

```perl
use Config::YAML;

my $c = Config::YAML->new(
             config => 'yaml_dump.txt',
             param1 => ['item1', 'item2', 'item3'],
             param2 => 'value2',
             paramN => {'paramN1' => 'valueN1'},
        );
my @list = qw(text1 text2);
$c->set_param4(\@list);
# Записываем структуру в файл
$c->write;
```

Содержимое файла yaml_dump.txt после записи:

```yaml
---
param1:
  - item1
  - item2
  - item3
param2: value2
param4:
  - text1
  - text2
paramN:
  paramN1: valueN1
```
