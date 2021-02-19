# Взаимодействие Catalyst-приложения с внешним API. Пример 1

*В дальнейшем хотелось бы найти время и рассмотреть несколько примеров взаимодействия Catalyst-приложения с внешними API. Например, API социальных сетей и авторизация с помощью OAuth (особенно заинтересовал Twitter), API платежных систем, API для совершения покупок он-лайн. Ниже - самый простой, односторонний, пример взаимодействия с внешней системой (строго говоря, это даже назвать API нельзя).*

Приведенный код строится на ранее созданной основе: <a href="http://dev-lab.info/2013/11/как-создать-catalyst-приложение-с-нуля">http://dev-lab.info/2013/11/как-создать-catalyst-приложение-с-нуля</a>

**Задача:**

Имеется интернет-магазин. В специальной директории, на сайте магазина, хранится несколько файлов с упорядоченной информацией для партнеров магазина. Все данные - в xml-формате. Списки городов, в которых осуществляет деятельность магазин, списки магазинов, списки товаров и т.п. Требуется просто организовать доступ к данным из Catalyst-приложения.

**Решение:**

Создаем модуль <font color="#00aa00">NorthMarket.pm</font>, который скачивает файлы, делает базовый парсинг данных. Создаем модуль <font color="#00aa00">Manage.pm</font>, который получает данные с помощью <font color="#00aa00">NorthMarket.pm</font> и оперирует ими по собственному усмотрению.

В данном случае, данные запрашиваются при обращении к странице <font color="#00aa00">http://localhost:3000/manage</font> . В дальнейшем, конечно же, можно организовать более интеллектуальный интерфейс - данные будут запрашиваться по нажатию кнопки, или можно вынести запрос в отдельный скрипт, который запускается по <font color="#00aa00">cron</font> и сохраняет все полученные данные в БД, или в виде файла. Но в указанном примере, я просто запрашиваю данные и вывожу на страничке список городов.

*/app/lib/Net/Source/NorthMarket.pm* :

```perl
package Net::Source::NorthMarket;

use Moose;
use XML::Simple;
use URI;
use LWP::UserAgent;
use XML::LibXML;
use Encode;

has host => (
    is => 'ro',
    isa => 'Str',
    default => 'http://northmarket.com/',
    required => 1,
);

sub get_cities {
    my ( $self ) = @_;

    my $url = URI->new( $self->host );
    $url->path_query('for_partner/city.xml');

    my $xml_out = $self->_call('GET', $url->as_string );
    return $self->_parse_xml($xml_out) || undef;
}

sub _parse_xml {
    my ( $self, $xml ) = @_;

    my $hash = XMLin( $xml );

    return $hash;
}

sub _call {
    my ($self, $method, $uri, $body) = @_;

    my $ua = LWP::UserAgent->new;
    my $req = HTTP::Request->new($method => $uri);
    my $res = $ua->request($req);
    return $res->decoded_content;
}

1;
```

*/app/lib/app/Controller/Manage.pm* :

```perl
package app::Controller::Manage;

use Moose;
use namespace::autoclean;
use uni::perl ':dumper';

use Net::Source::NorthMarket;
use utf8;
use Encode;

BEGIN { extends 'Catalyst::Controller' }

sub index :Path('/manage') :Args(0) {
    my ( $self, $c ) = @_;

    my $market = Net::Source::NorthMarket->new;
    my $res = $market->get_cities();

    my @cities;
    foreach (keys %{$res->{cities}->{city}}) {
        push @cities, encode('utf8', $_);
    }

    $c->stash->{cities} = \@cities;
    $c->stash->{template} = 'manage.tt';
}

__PACKAGE__->meta->make_immutable;

1;
```

*/app/root/src/manage.tt* :

```html
[% cities.join(', ') %]
```

Пример xml-файла для вышеприведенного кода :

```xml
<!--?xml version="1.0" encoding="UTF-8"?-->
<root> 
<cities> 
<city id="1" name="New York"> </city> 
<city id="2" name="Seattle"> </city> 
<city id="3" name="Forks"> </city> 
<city id="4" name="Portland"> </city>
</cities>
</root>
```


