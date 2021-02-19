# Как скачивать файлы в Catalyst-приложении. Catalyst::View::Download

*Как организовать скачивание файлов в Catalyst-приложении. Работа с форматами csv, tsv, txt, xml в Catalyst. Использование модуля Catalyst::View::Download . Добавление собственных форматов файлов.*

Достаточно часто встречаются ситуации, когда необходимо позволить пользователю скачать файл с сайта. Например, прайс-листы в xls-формате, шаблоны договоров, отчеты о продажах в csv, бланки квитанций для платежей и т.п.

<font color="#00aa00">Catalyst::View::Download</font> - создает view, который позволяет скачивать данные в различных форматах. Поддерживается работа с форматами: <font color="#00aa00">txt</font>, <font color="#00aa00">csv</font>, <font color="#00aa00">xml</font> и <font color="#00aa00">html</font>.

**1. Устанавливаем модуль Catalyst::View::Download**
<pre>force install Catalyst::View::Download</pre>

**2. Создаем новое view**
<pre>$ perl script/app_create.pl view Download Download
 exists "C:\Documents and Settings\natalie\www\app\app\lib\app\View"
 exists "C:\Documents and Settings\natalie\www\app\app\t"
created "C:\Documents and Settings\natalie\www\app\app\lib\app\View\Download.pm"
created "C:\Documents and Settings\natalie\www\app\app\t\view_Download.t"
</pre>

**3. Используем новое view для скачивания данных в txt, xml, html и csv форматах.**

Файл в csv-формате <font color="#00aa00">Catalyst::View::Download</font> генерирует сам, достаточно передать ему подготовленный массив с данными.

*lib/app/Controller/Admin/PaymentReports.pm :*

```perl
package app::Controller::Admin::PaymentReports;
use Moose;
use namespace::autoclean;

use utf8;

BEGIN { extends 'Catalyst::Controller'; }

sub payment_reports_csv :Path('/admin/payment_reports_csv') :Args(0) {
    my ( $self, $c ) = @_;

    my @data;
    push(@data,['Наименование', 'Сумма','Дата продажи']);
    push(@data,['Коляска дет., арт.45666', '3200','12.09.2014']);
    push(@data,['Набор игрушек, арт.67687', '320','12.09.2014']);
    push(@data,['Набор для творчества, арт.23221', '150','12.09.2014']);    

    $c->stash->{'download'} = 'text/csv';
    $c->stash->{'outfile_name'} = 'paymentreport';
    $c->stash->{'csv'} = \@data;
    
    $c->forward('View::Download');
}

sub get_blank_txt :Path('/admin/get_blank_txt') :Args(0) {
    my ( $self, $c ) = @_;

    my $blank = <<eof; Квитанция="" №="" "____"___________________20__г.="" Учреждение___________________________________
="" _____________________________________________="" Место="" нахождения_____________________________
="" Принято="" от___________________________________="" (фио)="" В="" уплату_____________________________________
="" (вид="" продукции,="" услуги,="" работы)="" Сумма,="" всего_________________________________
="" _________________________________руб.____коп.="" Получил____________="" ___________="" _____________
="" (должность)="" (подпись)="" (расшифровка)="" Уплатил__________="" "____"___________20__г.
="" eof="" $c-="">stash->{'download'} = 'text/plain';
    $c->stash->{'outfile_name'} = 'paymentblank';
    $c->stash->{'plain'} = $blank;
    
    $c->forward('View::Download');
}

sub get_blank_html :Path('/admin/get_blank_html') :Args(0) {
    my ( $self, $c ) = @_;

    my $blank = qq{
    <b>Квитанция №</b>};

    $c->stash->{'download'} = 'text/html';
    $c->stash->{'outfile_name'} = 'paymentblank';
    $c->stash->{'html'} = $blank;
    
    $c->forward('View::Download');
}

sub get_price_xml :Path('/admin/get_price_xml') :Args(0) {
    my ( $self, $c ) = @_;

    my $blank = qq{
<apirequest xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xs="http://www.w3.org/2001/XMLSchema">
   <apikey>76544999</apikey>
   <data>
     <row>
       <name>Position 1</name>
       <cost>45.33$</cost>
     </row>
     <row>
       <name>Position 2</name>
       <cost>22.80$</cost>
     </row>
   </data>
 </apirequest>
};
    $c->stash->{'download'} = 'text/xml';
    $c->stash->{'outfile_name'} = 'price';
    $c->stash->{'xml'} = $blank;
    
    $c->forward('View::Download');
}

__PACKAGE__->meta->make_immutable;

1;
```

**4. Запускаем сервер**

Вводим в браузере <font color="#00aa00">http://localhost:3000/admin/payment_reports_csv</font> .

Нам будет предложено сохранить <font color="#00aa00">paymentreport.csv</font> файл , или сразу открыть его с помощью какой-нибудь программы.

Для получения файлов с данными в других форматах, надо использовать адреса:
<ul>
<li><font color="#00aa00">http://localhost:3000/admin/get_price_xml</font> - файл с именем <font color="#00aa00">price.xml</font></li>
<li><font color="#00aa00">http://localhost:3000/admin/get_blank_html</font> - файл с именем <font color="#00aa00">paymentblank.html</font></li>
<li><font color="#00aa00">http://localhost:3000/admin/get_blank_txt</font> - файл с именем <font color="#00aa00">paymentblank.txt</font></li>
</ul>

## Добавление собственных форматов файлов

Допустим, мы хотим использовать <font color="#00aa00">Catalyst::View::Download</font> для того, чтобы пользователи могли скачивать файлы в форматах, отличных от предлагаемых.
Например, часто требуется отдавать данные в <font color="#00aa00">pdf</font> или <font color="#00aa00">doc</font> форматах.

Добавить поддержку новых форматов в <font color="#00aa00">Catalyst::View::Download</font> можно самостоятельно. Для этого достаточно немного изучить исходный код модулей <font color="#00aa00">Catalyst::View::Download</font> и <font color="#00aa00">Catalyst::View::Download::Plain</font>.

Для того, чтобы позволить клиенту скачивать файлы в формате doc, создаем модуль
<font color="#00aa00">app::View::Download::Doc</font>. Содержимое можно просто скопировать из модуля <font color="#00aa00">Catalyst::View::Download::Plain</font>, а вставки plain заменить на doc. "Content-Type" указать соответствующий для doc-файлов (можно посмотреть в таблицах mime-типов).

Созданный модуль позволит скачивать файлы doc-файлы. Doc-файл может быть создан заранее с помощью OpenOffice (и т.п.), либо генерироваться отдельными методами catalyst-приложения.

*lib/app/View/Download/Doc.pm:*

```perl
package app::View::Download::Doc;

use Moose;
use namespace::autoclean;

extends 'Catalyst::View';

our $VERSION = "0.01";
$VERSION = eval $VERSION;

__PACKAGE__->config( 'stash_key' => 'doc' );

sub process {
    my $self = shift;
    my ($c) = @_;

    my $template = $c->stash->{ 'template' };
    my $content  = $self->render( $c, $template, $c->stash );

    $c->res->headers->header( "Content-Type" => "application/vnd.
openxmlformats-officedocument.wordprocessingml.document" )
        if ( $c->res->headers->header( "Content-Type" ) eq "" );
    $c->res->body( $content );
}

sub render {
    my $self = shift;
    my ( $c, $template, $args ) = @_;

    my $stash_key = $self->config->{ 'stash_key' };
    my $content   = $c->stash->{ $stash_key } || $c->response->body;

    return $content;
}

__PACKAGE__->meta->make_immutable;

1;
```

После того, как создан модуль <font color="#00aa00">app::View::Download::Doc</font>, прописываем его в конфиге модуля <font color="#00aa00">Catalyst::View::Download</font> .

*lib/app/View/Download.pm :*

```perl
package app::View::Download;
use Moose;
use namespace::autoclean;

extends 'Catalyst::View::Download';

__PACKAGE__->config->{'content_type'}{'application/vnd.openxmlformats-
officedocument.wordprocessingml.document'} = {
    outfile_ext => 'doc',
    module      => '+Download::Doc'
};

__PACKAGE__->meta->make_immutable;

1;
```

После этого создаем модуль, или просто подпрограмму, которая будет обрабатывать запрос от пользователя, на скачивание doc-файла. При необходимости, можно генерить файл самостоятельно, в рамках catalyst-приложения, но в данном примере, мы просто берем готовый doc-файл, считываем его и передаем клиенту.

*lib/app/Controller/Admin/Partner.pm :*

```perl
package app::Controller::Admin::Partner;
use Moose;
use namespace::autoclean;

BEGIN { extends 'Catalyst::Controller'; }

sub get_oferta :Path('/admin/partner/get_oferta') :Args(0) {
    my ( $self, $c ) = @_;

    my $filename = '/oferta.doc';
    open(my $fh, '<', $filename) or die "Can't open file: $!";
    binmode($fh);
    local $/;
    my $content = <$fh>;
    close($fh);
 
    $c->stash->{'download'} = 'application/vnd.openxmlformats-office
document.wordprocessingml.document';
    $c->stash->{'outfile_name'} = 'oferta_for_partners';
    $c->stash->{'doc'} = $content;

    $c->forward('View::Download');
}

__PACKAGE__->meta->make_immutable;

1;
```

После того, как пользователь введет в браузере <font color="#00aa00">http://localhost:3000/admin/partner/get_oferta</font>, ему будет предложено сохранить файл под именем <font color="#00aa00">oferta_for_partners.doc</font> .

