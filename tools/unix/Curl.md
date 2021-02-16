# Команды Curl для отправки запросов методами GET, PUT, POST, DELETE

<i>Curl бывает полезен, когда надо тестировать сервис, работающий на основе REST.</i>

Вместо *url* подставьте нужный вам адрес, например: *"http://dev-lab.info/api/articles"*.

## GET

<pre>curl "uri?key1=value1&amp;key2=value2"</pre>

## POST

<pre>curl -d "key1=value1&amp;key2=value2" "uri"</pre>

## PUT

В данном случае, возможны 2 ситуации.

**Надо отправить полноценный PUT запрос.** Создаем файл *filename*, сохраняем в нем строку с данными:
<pre>key1=value1&amp;key2=value2</pre>
Потом выполняем запрос:
<pre>curl -T filename "uri"</pre>

**Надо отправить запрос методом PUT, но метод, это единственное, что отличает его
от запроса GET - по форме и содержанию**. В этом случае, тоже создаем файл, но оставляем его полностью пустым.
А потом выполняем уже указанную выше команду.

## DELETE

<pre>curl -X DELETE "uri"</pre>

Пример:
<pre>curl -u natalie:mypass -c cookie.txt -b cookie.txt -X DELETE http://dev-lab.info/api/articles/45</pre>
Ответ:
<pre>HTTP/1.1 200 OK
Server: nginx/1.0.3
Date: Thu, 27 Dec 2012 10:00:08 GMT
Content-Type: application/json
Connection: keep-alive
Vary: Content-Type
Content-Length: 2
Set-Cookie: sid=99c14ab8495958586fa06ae60d5ecaaaef13f23c; path=/; expires=Sun, 27-Dec
-2012 10:00:08 GMT; HttpOnly
Status: 200

{}
</pre>
