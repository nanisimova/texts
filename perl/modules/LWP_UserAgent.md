# Самый простой http-клиент на perl

```perl
#!/usr/bin/perl

use strict;

use LWP::UserAgent;

my $request = HTTP::Request->new(GET => 'http://localhost:32080/');
$request->header('Content-Type' => 'text/xml');

my $ua = LWP::UserAgent->new;
my $response = $ua->request($request);

print $response->content;
```

Очень лениво набирать этот код раз за разом. Добавляю эти несколько строк кода для дальнейшего копипаста. Ведь постоянно нужно что-то тестировать, и простой http-клиент, особенно, если надо тестировать взаимодействие с партнерами посредством обмена xml-данными, бывает незаменим.


