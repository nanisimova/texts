# Функции Perl для работы с сокетами

Небольшая заметка. Что такое сокет. Функции accept, bind, connect, getpeername, getsockname, getsockopt, listen, recv, send, setsockopt, shutdown, socket, socketpair. Простые примеры кода сервера и клиента.

## Что такое сокет. Краткое описание логики работы сокета

Сокеты Беркли — это API, библиотека для разработки приложений на языке Си, которая позволяет организовать межсетевое взаимодействие.

Альтернативой сокетам Беркли, является API TLI (Transport Level Interface), основанный на STREAMS.

Базовые операции сокетов для TCP: SOCKET, BIND, LISTEN, ACCEPT, SEND, RECEIVE, CLOSE.

Операция SOCKET создает новый сокет и выделяет для него место в таблице транспортной подсистемы. Параметры вызова указывают используемый формат адресов, тип требуемого сервиса и протокол. В случае успеха возвращает обычный файловый дескриптор, который можно использовать в дальнейших операциях.

Только что созданный сокет не связан ни с какими сетевыми адресами. Сетевой адрес назначается с помощью операции BIND.

Следующая операция - LISTEN, которая выделяет место для очереди входящих соединений.
Очередь нужна на тот случай, если единовременно к серверу обращается сразу несколько клиентов. Кроме того, LISTEN заявляет о готовности сервера принимать соединения.

ACCEPT - пассивное установление соединения. В противоположность ему, могут использоваться активные попытки установить соединение - операция CONNECT. Т.е. разница в том, кто именно инициирует попытки установить связь. После того, как с помощью ACCEPT был получен запрос на соединение, транспортная подсистема создает новый сокет с теми же параметрами, как у исходного, и возвращает дескриптор файла для него. На этом этапе, для обработки поступившего запроса, может быть создан дочерний процесс, который займется обработкой, в то время как исходный сокет вернется к прослушиванию порта и ожиданию новых запросов.

Для отправки и получения данных между клиентом и сервером, после установления соединения, можно использовать операции SEND и RECEIVE. В модели сокетов используется симметричный разрыв соединения, т.е. соединение будет разорвано после того, как обе стороны выполнят операцию CLOSE.


## Функции Perl для работы с сокетами

### socket

<pre>socket(SOCKET, DOMAIN, TYPE, PROTOCOL);</pre>

Функция создает сокет заданного типа и связывает его с файловым дескриптором SOCKET. До того, как будет позвана функция socket(), необходимо подключить модуль Socket:

<pre>use Socket;</pre>

Этот модуль поставляет необходимые для работы с сокетами константы. Функция возвращает значение true в случае успешного выполнения.

DOMAIN выбирает семейство протоколов, которые будут использоваться сокетом для создания соединения:
<ul>
<li>PF_INET для сетевого протокола IPv4 или</li>
<li>PF_INET6 для IPv6.</li>
<li>PF_UNIX (PF_LOCAL) для локальных сокетов (используя файл).</li>
</ul>

TYPE один из:
<ul>
<li>SOCK_STREAM - Надёжная потокоориентированная служба (сервис) или потоковый сокет. Обеспечивает создание двусторонних надежных и последовательных потоков байтов , поддерживающих соединения. Может также поддерживаться механизм внепоточных данных.</li>
<li>SOCK_DGRAM - Датаграммный сокет, ненадежные сообщения с ограниченной длиной и не поддерживающие соединения.</li>
<li>SOCK_RAW - Обеспечивает доступ к низкоуровневому сетевому протоколу. Perl поддерживает работу с SOCK_RAW, при подключении дополнительных модулей.</li>
</ul>

Параметр PROTOCOL задает конкретный протокол, который работает с сокетом.

### socketpair

<pre>socketpair(SOCKET1, SOCKET2, DOMAIN, TYPE, PROTOCOL);</pre>

Эта функция создает пару сокетов заданного типа, в заданном домене. Функция возвращает true, в случае успеха и false - если выполнение по тем или иным причинам завершилось неудачно. В системах, где не реализован socketpair(2), вызов этой функции  приведет к возникновению исключительной ситуации.

Обычно эта функция вызывается непосредственно перед fork().

### bind

<pre>bind(SOCKET, NAME);</pre>

Функция bind связывает заранее известный порт и ip-адрес сервера с созданным сокетом. Функция возвращает true в случае успеха, или false - в противном случае.

NAME - это специальным образом упакованная структура, которая содержит ip-адрес и порт сервера.

### listen

<pre>listen(SOCKET, QUEUESIZE);</pre>

С помощью функции listen, созданный сокет преобразуется в прослушивающий сокет, и может начать принимать входящие совединения от клиентов. Кроме того, функция задает максимальное количество клиентских соединений, которые ядро ставит в очередь на прослушиваемом сокете.

Функция возвращает значение true, в случае успеха, и false - в противном случае.

### accept

<pre>accept(NEW_SOCKET, SOCKET);</pre>

При вызове accept, процесс сервера блокируется и переходит к ожиданию клиентского подключения. Как только подключение было установлено, функция accept возвращает значение дескриптора. С помощью этого нового дескриптора будет осуществляться связь с клиентом.

Для каждого клиента будет заводиться новый дескриптор.

### recv

<pre>recv(SOCKET, SCALAR, LEN, FLAGS);</pre>

Функция получает сообщение на сокет. Пытается получить LENGHT байт в переменную SCALAR из заданного файлового дескриптора SOCKET. Функция возвращает адрес отправителя или undef, в случае ошибки.

### send

<pre>send(SOCKET, MSG, FLAGS);</pre>

Функция посылает сообщение на сокет.

### shutdown

<pre>shutdown(SOCKET, HOW);</pre>

Функция закрывает соединение с сокетом. Если HOW имеет значение
0 - дальнейший прием данных запрещен,
1 - запрещена дальнейшая отправка данных,
2 - запрещены и прием, и отправка данных.

Функция используется, когда неободимо сообщить другой стороне, что закончена запись, но еще не закончено чтение, или наоборот.

Функция отключает копии дескрипторов соединения в дочерних процессах.

### close

<pre>close(SOCKET);</pre>

Функция закрывает сокет. Возвращает значение true, если закрытие прошло успешно, и false - в противном случае.

### getpeername

<pre>getpeername(SOCKET);</pre>

Функция возвращает упакованный адрес сокета противоположного конца соединения SOCKET.

### getsockname

<pre>getsockname(SOCKET);</pre>

Функция возвращает упакованный адрес сокета данного конца соединения SOCKET. Это может пригодиться, если дескриптор сокета был передан нам от родительского процесса, и мы не знаем исходных параметров создания сокета. Или в иных, подобных случаях.

### getsockopt

<pre>getsockopt(SOCKET, LEVEL, OPTNAME);</pre>

Функция возвращает значение запрошенной опции сокета, или undef, в случае ошибки.

### setsockopt

<pre>setsockopt(SOCKET, LEVEL, OPTNAME, OPTVAL);</pre>

Функция устанавливает требуемые параметры сокета. Возвращает undef в случае ошибки.
LEVEL - обозначает уровень протокола, на который нацелен вызов. Можно указать SOL_SOCKET, если параметры задаются для самого сокета, поверх всех уровней.

## Функции модуля Socket.pm

### inet_aton

<pre>$ip_address = inet_aton($string);</pre>

Функция inet_aton выполняет преобразование ip-адреса из обычной строки в упакованный бинарный формат. Если преобразование выполнить не удалось, функция вернет undef. Работает только с адресами типа IPv4.

### inet_ntoa

<pre>$string = inet_ntoa($ip_address);</pre>

Функция inet_ntoa - выполняет обратное inet_aton преобразование. Принимает на вход упакованный адрес, возвращает i-адрес в виде обычной ascii-строки. Только для адресов IPv4.

### inet_pton

<pre>$address = inet_pton($family, $string);</pre>

Принимает на вход название семейства адресов (например, PF_INET6), строку с адресом и возвращает указанный адрес в упакованном бинарном виде. Т.е. альтернатива inet_aton, с большими возможностями.

### inet_ntop

<pre>$string = inet_ntop($family, $address);</pre>

Функция принимает на вход - название семейства, к которому принадлежит адрес, ссылку на упакованный адрес и возвращает ip-адрес в виде обычной строки. Функция является более современной альтернативой inet_ntoa.

### pack_sockaddr_in

<pre>$sockaddr = pack_sockaddr_in($port, $ip_address);</pre>

Функция принимает на вход два аргумента, номер порта и уже упакованный с помощью inet_aton, ip-адрес сервера. Возвращает упакованную структуру, которую в дальнейшем используют в командах bind, connect и send.

## Пример  работы с сокетами 1

Код сервера:

```perl
#!/usr/bin/perl -w

use strict;
use Socket;

my $port = 3002;
my $proto = getprotobyname('tcp');
my $server = "195.20.224.234";

socket(SOCKET, PF_INET, SOCK_STREAM, $proto) 
  or die "Can't open socket $!\n";
setsockopt(SOCKET, SOL_SOCKET, SO_REUSEADDR, 1) 
  or die "Can't set socket option to SO_REUSEADDR $!\n";

bind( SOCKET, pack_sockaddr_in($port, inet_aton($server)))
  or die "Can't bind to port $port! \n";

listen(SOCKET, 5) or die "listen: $!";
print "SERVER started on port $port\n";

my $client_addr;
while ($client_addr = accept(NEW_SOCKET, SOCKET)) {
   my $name = gethostbyaddr($client_addr, AF_INET );
   print NEW_SOCKET "Smile from the server";
   print "Connection recieved from $name\n";
   close NEW_SOCKET;
}

close SOCKET;
```

Код клиента:

```perl
use strict;
use Socket;

my $port = 3002;
my $server = "195.50.224.234";  

socket(SOCKET,PF_INET,SOCK_STREAM,(getprotobyname('tcp'))[2])
  or die "Can't create a socket $!\n";
connect( SOCKET, pack_sockaddr_in($port, inet_aton($server)))
  or die "Can't connect to port $port! \n";

my $line;
while ($line = <socket>) {
        print "$line\n";
}

close SOCKET or die "close: $!";
```

При обращении клиента, сервер вывел в STDOUT:
<pre>
$ perl socket2.pl
SERVER started on port 3002
Connection recieved from ANantes-651-1-63-199.w2-0.abo.wanadoo.fr
</pre>

Результат работы клиента:
<pre>&gt; perl socket_client2.pl
Smile from the server
</pre>

## Пример работы с сокетами 2

Создание клиента и сервера, работа с сокетами, использование модуля IO::Socket::INET.

Программный код сервера:

```perl
use strict;
use IO::Socket::INET;

my $server = IO::Socket::INET->new(
  LocalPort => '3002',
  Type => SOCK_STREAM,
  Reuse => 1,
  Listen => 2
) or die "Не могу стать сервером на порту 3002: $!\n";

while (my $client = $server->accept()) {

    my $client_address = $client->peerhost();
    my $client_port = $client->peerport();
    print "connection from $client_address:$client_port\n";

    my $data = "";
    $client->recv($data, 1024);
    print "received data: $data\n";

    # отправляем ответ клиенту
    $data = "ok";
    $client->send($data);

    # уведомляем клиента, что отправка данных завершена
    shutdown($client, 1);
}
close($server);
```

Сервер доступен по адресу: http://195.50.224.234:3002/ Для обращения к нему можно использовать браузер.

При обращении через браузер, сервер выдает в STDOUT:
<pre>
$ perl socket.pl
connection from 91.79.9.111:33006
received data: GET / HTTP/1.1
Host: 195.50.224.234:3002
User-Agent: Mozilla/5.0 (Windows NT 5.1; rv:35.0) Gecko/20100101 Firefox/35.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: ru-RU,ru;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: keep-alive
</pre>

Код клиента:

```perl
use strict;
use IO::Socket::INET;
 
$| = 1;
 
my $socket = IO::Socket::INET->new(
    PeerHost => '195.50.224.234',
    PeerPort => '3002',
    Proto => 'tcp',
);
die "cannot connect to the server $!\n" unless $socket;
print "connected to the server\n";
 
my $req = 'hello world';
my $size = $socket->send($req);
print "sent data of length $size\n";
 
shutdown($socket, 1);
 
my $response = "";
$socket->recv($response, 1024);
print "received response: $response\n";
 
$socket->close();
```

При запуске клиента, сервер выведет в STDOUT:
<pre>
connection from 91.79.9.111:33030
received data: hello world
</pre>

Клиент в свою очередь выведет:

<pre>
$ perl socket_client.pl
connected to the server
sent data of length 11
received response: ok
</pre>

В этом примере, клиент подключается к серверу, используя известной ip-адрес и порт. После того, как клиент подключился к серверу, он посылает данные на сервер и ожидает ответа. Сервер, после получения данных от клиента, отвечает специальным сообщением.

Shutdown(SOCKET, 1) в примере используется, чтобы, не закрывая сокет, сообщить системе на другом конце соединения, о завершении передачи данных.

Используем netstat для получения информации о текущем состоянии нашей системы:

Запрос 1:
<pre>
$ netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 *:ssh                   *:*                     LISTEN
tcp        0      0 195-50-224-234.ovz.:ssh ppp91-79-9-111.pp:58219 ESTABLISHED
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     102647   @/com/ubuntu/upstart
unix  2      [ ACC ]     SEQPACKET  LISTENING     102648   @run/udev/control
unix  2      [ ]         DGRAM                    102660
unix  3      [ ]         DGRAM                    102659
unix  3      [ ]         DGRAM                    102658
</pre>

Запрос 2:
<pre>
$ netstat -n
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 195.50.224.234:22       91.79.9.111:58219       ESTABLISHED
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ]         DGRAM                    102660
unix  3      [ ]         DGRAM                    102659
unix  3      [ ]         DGRAM                    102658
</pre>

Запрос 3:
<pre>
$ netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         *               0.0.0.0         U         0 0          0 venet0
</pre>

После того, как был запущен сервер, значения netstat изменились:
<pre>
$ netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 *:ssh                   *:*                     LISTEN
tcp        0      0 *:3002                  *:*                     LISTEN
tcp        0      0 195-50-224-234.ovz.:ssh ppp91-79-9-111.pp:33309 ESTABLISHED
tcp        0      0 195-50-224-234.ovz.:ssh ppp91-79-9-111.pp:58219 ESTABLISHED
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     102647   @/com/ubuntu/upstart
unix  2      [ ACC ]     SEQPACKET  LISTENING     102648   @run/udev/control
unix  2      [ ]         DGRAM                    102660
unix  3      [ ]         DGRAM                    102659
unix  3      [ ]         DGRAM                    102658
</pre>

Запрос 2:
<pre>
$ netstat -ni
Kernel Interface table
Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
lo        16436 0    199864      0      0 0        199864      0      0      0 LRU
venet0     1500 0   7514376      0      0 0       8632984      0      0      0 BOPRU
venet0:0   1500 0       - no statistics available -                        BOPRU
</pre>

Запрос 3:
<pre>
$ ifconfig lo
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:199864 errors:0 dropped:0 overruns:0 frame:0
          TX packets:199864 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:780861518 (744.6 MiB)  TX bytes:780861518 (744.6 MiB)
</pre>

Запрос 4:
<pre>
$ ifconfig venet0:0
venet0:0  Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00
          inet addr:195.50.224.234  P-t-P:195.50.224.234  Bcast:195.50.224.234  Mask:255.255.255.255
          UP BROADCAST POINTOPOINT RUNNING NOARP  MTU:1500  Metric:1
</pre>
Узнаем номер процесса сервера:
<pre>
$ ps -u root -f
root     29832 29827  0 07:01 pts/2    00:00:00 perl socket.pl
</pre>
В списке файлов, открытых процессом сервера, появилась строка с информацией о сокете:
<pre>
$ lsof -a -p 29832
COMMAND   PID USER   FD   TYPE     DEVICE SIZE/OFF       NODE NAME
perl    29832 root  cwd    DIR    144,108     4096    2564131 /home/aninatalie/diff
perl    29832 root  rtd    DIR    144,108     4096    2528296 /
perl    29832 root  txt    REG    144,108  1487332    4489619 /usr/bin/perl
perl    29832 root  mem    REG        8,3             4489619 /usr/bin/perl (path dev=144,108)
...
perl    29832 root    0u   CHR      136,2      0t0          5 /dev/pts/2
perl    29832 root    1u   CHR      136,2      0t0          5 /dev/pts/2
perl    29832 root    2u   CHR      136,2      0t0          5 /dev/pts/2
perl    29832 root    3u  IPv4 1022538664      0t0        TCP *:3002 (LISTEN)
perl    29832 root    7w  FIFO        0,8      0t0 1022529722 pipe
</pre>

