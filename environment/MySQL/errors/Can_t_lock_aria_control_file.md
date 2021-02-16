# Ошибка mysql "Can’t lock aria control file"

*Ошибка mysql "Can't lock aria control file '/var/lib/mysql/aria_log_control' for exclusive use". Вариант решения проблемы.*


Сегодня столкнулась с хитрой проблемой mysql, о которой не могу не сделать запись. Материалов в сети по этой проблеме я нашла не много, и из тех, которые нашла - мне практически ничего не помогло. Поэтому, считаю необходимым поделиться - вдруг кому-то пригодится.

**Проблема:** после сбоя в питании у меня зависла виртуальная машина с окружением для разработки. После полной перезагрузки системы, включая базовую ОС, виртуальная машина заработала. А вот mysql запускаться категорически отказался.
<pre>$ sudo /etc/init.d/mysql start
 Starting MariaDB database server mysqld                  [fail]
</pre>

В логах (<font color="#00aa00">*/var/log/mysql/error.log*</font>) можно было увидеть ошибку
**"Can't lock aria control file '/var/lib/mysql/aria_log_control' for exclusive use"** :

<pre>150925 17:09:40 mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
150925 17:16:49 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
150925 17:16:49 [Note] /usr/sbin/mysqld (mysqld 5.5.44-MariaDB-1ubuntu0.14.04.1) 
starting as process 8926 ...
150925 17:16:49 [ERROR] mysqld: Can't lock aria control file '/var/lib/mysql
/aria_log_control' for exclusive use, error: 11. Will
 retry for 30 seconds
...
150925 17:17:22 [ERROR] Aborting

150925 17:17:22  InnoDB: Starting shutdown...
150925 17:17:22  InnoDB: Shutdown completed; log sequence number 5303048825
150925 17:17:22 [Note] /usr/sbin/mysqld: Shutdown complete
</pre>
Не буду описывать все, что я пробовала сделать. Скажу сразу готовое решение.

В директории <font color="#00aa00">*/var/lib/mysql*</font> удаляем два файла:
<font color="#00aa00">*aria_log.00000001*</font> и <font color="#00aa00">*aria_log_control*</font>.
Если страшно - можно просто переименовать, например, в <font color="#00aa00">*old_aria_log.00000001*</font> и
<font color="#00aa00">*old_aria_log_control*</font>.

После этого запустила сервер, все заработало.
<pre>$ sudo /etc/init.d/mysql start
 Starting MariaDB database server mysqld                                [ OK ]
 Checking for corrupt, not cleanly closed and upgrade needing tables.
</pre>
Возможно, понадобится проверить права доступа к файлам, чтобы mysql смог к ним обращаться. Но это зависит от настроек
вашей системы.


