# Функции perl для получения информации о сети

*Функции: endhostent, endnetent, endprotoent, endservent, gethostbyaddr, gethostbyname, gethostent, getnetbyaddr, getnetbyname, getnetent, getprotobyname, getprotobynumber, getprotoent, getservbyname, getservbyport, getservent, sethostent, setnetent, setprotoent, setservent. Примеры кода.*

## Функции для получения информации о хостах

### endhostent

<pre>endhostent</pre>
Закрытие файла хостов после завершения обработки его содержимого.

### gethostbyaddr

<pre>gethostbyaddr(ADDR, ADDRTYPE);</pre>
Возвращает описание интернет-узла по заданному сетевому адресу.

<font color="#00aa00">ADDR</font> - это упакованный бинарный сетевой адрес, <font color="#00aa00">ADDRTYPE</font> - тип, семейство протоколов.

Информацию о хосте получает от сервера имен или из файла <font color="#00aa00">/etc/hosts </font>.

Формат вызова:
<pre>($name, $aliases, $addrtype, $length, @addrs) = gethostbyaddr($address, $addrtype);</pre>

Просматриваем <font color="#00aa00">/etc/hosts</font> на указанном сервере:
<pre>
fe00::0         ip6-localnet
ff00::0         ip6-mcastprefix
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.0.1 localhost.localdomain localhost
# Auto-generated hostname. Please do not remove this comment.
179.20.224.220 179-20-224-220.ovz.vps.regruhosting.ru  179-20-224-220
::1             localhost ip6-localhost ip6-loopback
</pre>

Пример:
```perl
use strict;
use Socket;

my ($name, $aliases, $addrtype, $length, @addrs) = gethostbyaddr(inet_aton("179.20.224.220"), PF_INET);
print "NAME ".$name."\n";
print "ALIASES ".$aliases."\n";
print "ADDRTYPE ".$addrtype."\n";
print "LENGHT ".$length."\n";

print inet_ntoa($addrs[0])." ".scalar(@addrs)."\n";
```

Вывод:
<pre>
$ perl net1.pl
NAME 179-20-224-220.ovz.vps.regruhosting.ru
ALIASES 179-20-224-220
ADDRTYPE 2
LENGHT 4
179.20.224.220 1
</pre>

### gethostbyname

<pre>
gethostbyname(NAME);
($name, $aliases, $addtype, $length, @addrs) = gethostbyname($remote_hostname);
</pre>

Возвращает описание интернет-узла по указанному имени хоста.

Возвращается информация, полученная от сервера имен или произвольные поля из строки в <font color="#00aa00">/etc/hosts</font> . Если нет доступа к серверу имен, то будет просматриваться локальный <font color="#00aa00">/etc/hosts</font> .

Одному имени хоста может соответствовать несколько ip-адресов. Особенно часто это встречается в высоконагруженных проектах, где происходит распределение нагрузки по разным серверам.

Формат возвращаемых данных такой же, как для функции <font color="#00aa00">gethostbyaddr()</font>.

Пример:

```perl
use strict;
use Socket;

my ($name, $aliases, $addrtype, $length, @addrs) = gethostbyname("yandex.ru");

print "NAME ".$name."\n";
print "ALIASES ".$aliases."\n";
print "ADDRTUPE ".$addrtype."\n";
print "LENGHT ".$length."\n";

foreach (@addrs) {
  print inet_ntoa($_)."\n";
}
```

Вывод:
<pre>
$ perl net2.pl
NAME yandex.ru
ALIASES
ADDRTUPE 2
LENGHT 4
93.158.134.11
213.180.204.11
213.180.193.11
</pre>

### gethostent

<pre>gethostent</pre>

Функция осуществляет итерацию по файлу <font color="#00aa00">/etc/hosts</font> и возвращает по одной записи. Данные возвращаются в том же формате, что и для функции <font color="#00aa00">gethostbyaddr()</font>.

Функция <font color="#00aa00">gethostent()</font> может быть не реализована для некоторых видов операционных систем, так что, скрипты, которые содержат ее - не могут считаться переносимыми.
<pre>
$ less /etc/hosts
fe00::0         ip6-localnet
ff00::0         ip6-mcastprefix
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.0.1 localhost.localdomain localhost
# Auto-generated hostname. Please do not remove this comment.
179.20.224.220 179-20-224-220.ovz.vps.regruhosting.ru  179-20-224-220
::1             localhost ip6-localhost ip6-loopback
</pre>

Пример:

```perl
use strict;
use Socket;

while (my ($name, $aliases, $addrtype, $length, @addrs) = gethostent) {

  print "NAME ".$name."\n";
  print "ALIASES ".$aliases."\n";
  print "ADDRTUPE ".$addrtype."\n";
  print "LENGHT ".$length."\n";

  foreach (@addrs) {
    print inet_ntoa($_)."\n";
  }
}
```

Вывод:
<pre>
$ perl net3.pl
NAME localhost.localdomain
ALIASES localhost
ADDRTUPE 2
LENGHT 4
127.0.0.1
NAME 179-20-224-220.ovz.vps.regruhosting.ru
ALIASES 179-20-224-220
ADDRTUPE 2
LENGHT 4
179.20.224.220
NAME localhost
ALIASES ip6-localhost ip6-loopback
ADDRTUPE 2
LENGHT 4
127.0.0.1
</pre>

### sethostent

<pre>sethostent(STAYOPEN);</pre>
Позиционирование файла хостов на первую запись. Последующий вызов функции <font color="#00aa00">gethostent()</font> начинает его обработку с первой записи.

Если <font color="#00aa00">STAYOPEN</font> указан как <font color="#00aa00">true</font>, то файл не будет закрываться между вызовами <font color="#00aa00">gethostbyname()</font> и
<font color="#00aa00">gethostbyaddr()</font>.

## Функции для получения информации о сетях

### endnetent

<pre>endnetent</pre>
Закрытие файла сетей после завершения обработки его содержимого.

### getnetbyaddr

<pre>getnetbyaddr(ADDR, ADDRTYPE);</pre>

Получение записи из файла сетей о сети с заданным адресом и типом.
<pre>($name, $aliases, $addrstype, $net) = getnetbyaddr($netnumber, $type);</pre>

В скалярном контексте функция вернет только сетевое имя.

Пример:

```perl
use strict;
use Socket;

my ($name, $aliases, $addrstype, $net) = getnetbyaddr(0, PF_INET);

print $name."\n";
print $aliases."\n";
print $addrstype."\n";
print $net."\n";
```

Вывод:
<pre>
$ perl net4.pl
default

2
0
</pre>

### getnetbyname

<pre>getnetbyname(NAME);</pre>
Получение записи из файла сетей о сети с заданным сетевым именем.
<pre>($name, $aliases, $addrstype, $net) = getnetbyname($netname);</pre>

Пример:

```perl
use strict;

my ($name, $aliases, $addrstype, $net) = getnetbyname("link-local");

print $name."\n";
print $aliases."\n";
print $addrstype."\n";
print $net."\n";
```

Вывод:
<pre>
$ perl net4.pl
link-local

2
2851995648
</pre>

### getnetent

<pre>getnetent</pre>
Возвращает следующую запись файла сетей при очередном вызове.

Функция выполняет итерацию по строкам файла <font color="#00aa00">/etc/networks</font>. В списковом контексте возвращает значения:
<pre>($name, $aliases, $addrtype, $net) = getnetent();</pre>

В скалярном контексте возвращает только имя сети.
<pre>$ less /etc/networks
default         0.0.0.0
loopback        127.0.0.0
link-local      169.254.0.0
</pre>

<pre>$ man -S 5 networks</pre>

Пример:

```perl
while (my ($name, $aliases, $addrtype, $net) = getnetent) {
  print "NAME ".$name."\n";
  print "ALIASES ".$aliases."\n";
  print "ADDRTYPE ".$addrtype."\n";
  print "NET ".$net."\n\n";
}
```

где <font color="#00aa00">$name</font> - официальное имя сети, <font color="#00aa00">$addrtype</font> - тип сетевого адреса, <font color="#00aa00">$net</font> - номер сети в сетевом порядке байтов.

Вывод:
<pre>
$ perl net3.pl
NAME default
ALIASES
ADDRTYPE 2
NET 0

NAME loopback
ALIASES
ADDRTYPE 2
NET 2130706432

NAME link-local
ALIASES
ADDRTYPE 2
NET 2851995648
</pre>

### setnetent

<pre>setnetent(STAYOPEN);</pre>
Позиционирование файла сетей на первую запись. Последующий вызов функции <font color="#00aa00">getnetent()</font> начинает его обработку с первой записи.

Если <font color="#00aa00">STAYOPEN</font> задан <font color="#00aa00">true</font>, то файл не будет закрываться между вызовами <font color="#00aa00">getnetbyname()</font> и <font color="#00aa00">getnetbyaddr()</font>.

## Функции для получения информации о протоколах

### endprotoent

<pre>endprotoent</pre>

Закрытие файла протоколов после завершения обработки его содержимого.

### getprotobyname

<pre>getnetbyname(NAME);</pre>

Получение записи из файла протоколов <font color="#00aa00">/etc/protocols</font> о протоколе с заданным именем.

В скалярном контексте возвращает только номер протокола. Списковый контекст:

<pre>($name, $aliases, $protocol_number) = getprotobyname($proto_name);</pre>

Пример:

```perl
use strict;

my ($name, $aliases, $protocol_number) = getprotobyname("tcp");

print $name."\n";
print $aliases."\n";
print $protocol_number."\n";
```

Вывод:
<pre>
$ perl net4.pl
tcp
TCP
6
</pre>

### getprotobynumber
<pre>getprotobynumber(NUMBER);</pre>

Получение записи из файла протоколов <font color="#00aa00">/etc/protocols</font> о протоколе с заданным номером. В списковом контексте функция возвращает:

<pre>($name, $aliases, $protocol_number) = getprotobynumber($number);</pre>

В скалярном контексте функция вернет только название протокола.

```perl
use strict;

my ($name, $aliases, $protocol_number) = getprotobynumber(1);

print $name."\n";
print $aliases."\n";
print $protocol_number."\n";
```

Вывод:
<pre>
$ perl net4.pl
icmp
ICMP
1
</pre>

### getprotoent

<pre>getprotoent</pre>

Возвращает следующую запись файла протоколов при очередном вызове.
<pre>
$ less /etc/protocols
ip      0       IP              # internet protocol, pseudo protocol number
icmp    1       ICMP            # internet control message protocol
...
tcp     6       TCP             # transmission control protocol
...
udp     17      UDP             # user datagram protocol
...
ipv6    41      IPv6            # Internet Protocol, version 6
...
</pre>
Функция осуществляет итерацию файла <font color="#00aa00">/etc/protocols</font> . В списковом контексте возвращает:

<pre>($name, $aliases, $protocolnumber) = getprotoent();</pre>

В скалярном контексте возвращает только имя протокола.

Пример:

```perl
use strict;

while (my ($name, $aliases, $protocolnumber) = getprotoent) {
  print "NAME ".$name."\n";
  print "ALIASES ".$aliases."\n";
  print "PROTO ".$protocolnumber."\n\n";
}
```

Вывод:
<pre>
$ perl net3.pl
NAME ip
ALIASES IP
PROTO 0

NAME icmp
ALIASES ICMP
PROTO 1

NAME igmp
ALIASES IGMP
PROTO 2
</pre>

### setprotoent

<pre>setprotoent(STAYOPEN);</pre>

Позиционирование файла протоколов на первую запись. Последующий вызов функции <font color="#00aa00">getprotoent()</font> начинает его обработку с первой записи.

Если <font color="#00aa00">STAYOPEN</font> равен <font color="#00aa00">true</font>, то файл не будет закрываться между вызовами <font color="#00aa00">getprotobyname()</font> и
<font color="#00aa00">getprotobyaddr()</font>.

## Функции для получения информации о сервисах

### endservent

<pre>endservent</pre>

Закрытие файла сервисов после завершения обработки его содержимого.

### getservbyname

<pre>getservbyname(NAME, PROTO);</pre>
Получение записи из файла <font color="#00aa00">/etc/services</font> о сервисе с заданным именем и протоколом.

В списковом контексте функция возвращает:
<pre>($name, $aliases, $port_number, $protocol_name) = getservbyname($serv_name, $proto_name);</pre>

В скалярном контексте <font color="#00aa00">getservbyname()</font> вернет только номер порта.

Пример:

```perl
use strict;

my ($name, $aliases, $port_number, $protocol_name) = getservbyname("postgresql", "tcp");

print $name."\n";
print $aliases."\n";
print $port_number."\n";
print $protocol_name."\n";
```

Вывод:
<pre>
$ perl net4.pl
postgresql
postgres
5432
tcp
</pre>

### getservbyport

<pre>getservbyport(PORT, PROTO);</pre>

Получение записи из файла <font color="#00aa00">/etc/services</font> о сервисе с заданным номером порта и протоколом. В скалярном контексте <font color="#00aa00">getservbyport()</font> вернет только имя сервиса. В списковом контексте возвращает:

<pre>($name, $aliases, $port_number, $protocol_name) = getservbyport($port, $protocol);</pre>

Пример:

```perl
use strict;

my ($name, $aliases, $port_number, $protocol_name) = getservbyport(80, "udp");

print $name."\n";
print $aliases."\n";
print $port_number."\n";
print $protocol_name."\n";
```

Вывод:
<pre>$ perl net4.pl
http

80
udp
</pre>

### getservent

<pre>getservent</pre>

Возвращает следующую запись файла сервисов <font color="#00aa00">/etc/services</font> при очередном вызове.
<pre>
$ less /etc/services
tcpmux          1/tcp                           # TCP port service multiplexer
echo            7/tcp
echo            7/udp
discard         9/tcp           sink null
discard         9/udp           sink null
...
http            80/tcp          www             # WorldWideWeb HTTP
http            80/udp                          # HyperText Transfer Protocol
...
</pre>
В списковом контексте функция <font color="#00aa00">getservent()</font> возвращает:

<pre>($name, $aliases, $port_number, $protocol_name) = getservent();</pre>

В скалярном контексте <font color="#00aa00">getservent()</font> будет возвращать только имя сервиса.

Пример:

```perl
use strict;

while (my ($name, $aliases, $port_number, $protocol_name) = getservent) {
  print "NAME ".$name."\n";
  print "ALIASES ".$aliases."\n";
  print "PORT ".$port_number."\n";
  print "PROTONAME ".$protocol_name."\n\n";
}
```

Вывод:
<pre>NAME tcpmux
ALIASES
PORT 1
PROTONAME tcp

NAME echo
ALIASES
PORT 7
PROTONAME tcp

NAME fido
ALIASES
PORT 60179
PROTONAME tcp
</pre>

### setservent

<pre>setservent</pre>

Позиционирование файла сервисов на первую запись. Последующий вызов функции <font color="#00aa00">getservent()</font> начинает его обработку с первой записи.

Если <font color="#00aa00">STAYOPEN</font> равен <font color="#00aa00">true</font>, то файл не будет закрываться между вызовами <font color="#00aa00">getservbyname()</font> и <font color="#00aa00">getservbyaddr()</font>.

