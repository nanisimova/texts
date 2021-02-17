# Шпаргалка по Git — основные команды, слияние веток, выписка веток с github

*Шпаргалка по git. Пошаговое руководство: как выполнить слияние веток в git, как создать новую ветку и репозиторий, как выписать ветку с github и т.п. Инструкции по git для начинающих.*

<font color="#00aa00">Git</font> - это распределенная система контроля версий. Это главное отличие <font color="#00aa00">git</font> от <font color="#00aa00">svn</font>. Каждый разработчик создает на своем компьютере отдельный, полноценный репозиторий.

В рамках этого репозитория можно делать все тоже самое, что и обычно в svn - создавать ветки, просматривать изменения, выполнять коммиты. Для того, чтобы можно было работать с удаленными репозиториями и обмениваться изменениями с другими разработчиками, в git есть две команды, не имеющие аналогов в svn - <font color="#00aa00">git push</font> и <font color="#00aa00">git pull</font>.

<font color="#00aa00">git push</font> - вливание локальных изменений в удаленный репозиторий. <font color="#00aa00">git pull</font> - вливание изменений из удаленного репозитория в локальный. Обмен данными обычно происходит с использованием протокола SSH.

Git поддерживают несколько крупных репозиториев - <font color="#00aa00">GitHub</font>, <font color="#00aa00">SourceForge</font>, <font color="#00aa00">BitBucket</font> и <font color="#00aa00">Google Code</font>. Удобно использовать один из них в качестве основного хранилища для корпоративных проектов.

<i>Ниже приведены инструкции по использованию git в различных ситуациях. Что делать, если нужно создать новый репозиторий,  или выписать ветку, и т.п. Я использую подобную шпаргалку для скоростного копипаста :) Чтобы не отвлекаться, когда голова занята сложными задачами. По мере создания новых инструкций, статья будет обновляться.</i>


## Пошаговые рекомендации

### Как выписать репозиторий с github

<ol>
<li>Создаем новую директорию для проекта <font color="#00aa00">project_name</font>, переходим в нее.</li>
<li>Выполняем команду:
<pre>git clone git@github.com:devlabuser/sharp.git ./
</pre>
<font color="#00aa00">"./"</font> означает, что создать репозиторий нужно в текущей директории.</li>
</ol>
Результат: каталог с выписанной веткой <font color="#00aa00">master</font>. Теперь можно создавать новые ветки, или выписывать с <font color="#00aa00">github</font> существующие.


### Как выписать ветку с github

С помощью команды <font color="#00aa00">"checkout"</font> можно выписать уже существующую ветку с github:
<pre>$ git checkout -b dev origin/dev
$ git checkout -b project_branch origin/project_branch
</pre>

Или так, что намного надежнее:
<pre>$ git checkout --track origin/production</pre>

Если команда не сработала, нужно попробовать выполнить обновление:
<pre>$ git remote update</pre>

Если вышеприведенные команды не сработали, выдали ошибку, и времени разбираться с ней нет, можно попробовать получить нужную ветку следующим способом:
<pre>git checkout -b project_branch
git pull origin project_branch
</pre>

Т.е. сначала мы создаем новую ветку, а затем вливаем в нее изменения из ветки на github.

### Как создать новую ветку в локальном репозитории

<ol>
<li>Создаем новую ветку в локальном репозитории:
<pre>$ git checkout -b dev
Switched to a new branch 'dev'
</pre>
</li>
<li>Публикуем ее на github:
<pre>$ git push origin dev
Total 0 (delta 0), reused 0 (delta 0)
To git@github.com:devlabuser/sharp.git
 * [new branch]      dev -&gt; dev
</pre>
</li>
</ol>

### Как переключиться на другую ветку в git

<pre>$ git checkout project2_branch</pre>

Если вы случайно удалили какой-то файл, можно извлечь его из хранилища:
<pre>$ git checkout readme.txt</pre>

### Как посмотреть список веток

Команда <font color="#00aa00">"branch"</font> позволяет посмотреть список веток в локальном репозитории. Текущая ветка будет помечена звездочкой:
<pre>$ git branch
* dev
  master
</pre>

### Как сделать commit

Создаем новую ветку, выполняем в ней нужные изменения.
<ol>
<li>Список всех измененных и добавленных файлов можно просмотреть командой:
<pre>$ git status
</pre>
</li>

<li>Подготавливаем коммит, добавляя в него файлы командой:
<pre>$ git add &lt;file1&gt; &lt;file2&gt; ...</pre>

Или удаляем устаревшие файлы:
<pre>$ git rm &lt;file1&gt; &lt;file2&gt; ...</pre>
</li>

<li>Выполняем коммит:
<pre>$ git commit -m 'Комментарий к коммиту'</pre>
</li>

<li>Как правило, в репозитории существует две основные ветки - dev и master. Dev - общая ветка разработчиков и тестировщиков. Именно в нее добавляются все новые разработки перед очередным релизом. Master - ветка для выкладки продукта на боевые сервера.

После коммита надо влить в нашу ветку изменения из ветки dev и master:
<pre>$ git pull origin dev 
$ git pull origin master
</pre>
Теперь наша ветка содержит изменения для проекта, и все последние изменения по другим задачам, которые успела внести команда.</li>

<li>Переключаемся на ветку dev:
<pre>$ git checkout dev</pre>
</li>

<li>Вливаем в dev изменения из ветки проекта:
<pre>$ git merge project_branch</pre>
</li>

<li>Заливаем последнюю версию ветки dev на удаленный сервер:
<pre>$ git push origin dev

Counting objects: 4, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 286 bytes, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:devlab/sharp.git
   d528335..9a452d9  dev -&gt; dev
</pre>
<font color="#00aa00">push</font> может не пройти, потому что удалённый <font color="#00aa00">origin/dev</font> обогнал локальную его копию.</li>
</ol>

### Как решить конфликт бинарных файлов

Допустим, при слиянии с другой веткой git выдал ошибку. Команда <font color="#00aa00">git status</font> возвращает информацию о конфликте:
<pre>$ git status
...
Unmerged paths:
  (use "git add &lt;file&gt;..." to mark resolution)

        both modified:      root/css/styles.css.gz

$ git diff root/css/styles.css.gz
diff --cc root/css/styles.css.gz
index 970c721,bc6d170..0000000
Binary files differ
</pre>

Конфликтный файл является бинарным (это могут быть архивные файлы, изображения и т.п.), и решение конфликта стандартным способом, с помощью редактирования - не возможно.

Чтобы решить такой конфликт, надо просто выбрать - какая версия файла будет использоваться: ваша или из вливаемой ветки. Чтобы использовать свой вариант файла, вводим команду:
<pre>git checkout --ours binary.dat
git add binary.dat
</pre>
Если мы выбираем версию из вливаемой ветки:
<pre>git checkout --theirs binary.dat
git add binary.dat
</pre>
"<font color="#00aa00">ours</font>" - от английского "наш", "<font color="#00aa00">theirs</font>" - от английского "их".

### Как посмотреть историю изменений

<font color="#00aa00">git log</font> - просмотр логов.
<pre>$ git log
commit 9a452d9cdbdb57e7e4f2b09f8ce2f776cd56657a
Author: DevLab User &lt;user@mail.ru&gt;
Date:   Wed Jul 31 18:35:47 2013 +0400

    first commit

commit d528335724dfc15461996ed9d44d74f23ce6a075
Author: DevLab User &lt;user@mail.ru&gt;
Date:   Wed Jul 31 06:24:57 2013 -0700

    Initial commit
</pre>

Вывод данных о каждом коммите в одну строку:
<pre>git log --pretty=oneline

9a452d9cdbdb57e7e4f2b09f8ce2f776cd56657a first commit
d528335724dfc15461996ed9d44d74f23ce6a075 Initial commit
</pre>
Для вывода информации <font color="#00aa00">git log</font> использует просмотрщик, указанный в конфиге репозитория.

Поиск по ключевому слову в комментариях к коммиту:
<pre>$ git log | grep -e "first"
    first commit
</pre>

Команда <font color="#00aa00">"git show"</font> позволяет просмотреть, какие именно изменения произошли в указанном коммите:
<pre>$ git show 99452d955bdb57e7e4f2b09f8ce2fbb6cd56377a

commit 99452d955bdb57e7e4f2b09f8ce2fbb6cd56377a
Author: DevLab User &lt;user@mail.ru&gt;
Date:   Wed Jul 31 18:35:47 2013 +0400

    first commit

diff --git a/readme.txt b/readme.txt
new file mode 100644
index 0000000..4add785
--- /dev/null
+++ b/readme.txt
@@ -0,0 +1 @@
+Text
\ No newline at end of file
</pre>

Можно посмотреть построчную информацию о последнем коммите, имя автора и хэш коммита:
<pre>$ git blame README.md

^d528335 (devlabuser 2013-07-31 06:24:57 -0700 1) sharp
^d528335 (devlabuser 2013-07-31 06:24:57 -0700 2) =====
</pre>
git annotate, выводит измененные строки и информацию о коммитах, где это произошло:
<pre>$ git annotate readme.txt
9a452d9c        (DevLab User      2013-07-31 18:35:47 +0400       1)Text
</pre>

### Как сделать откат

<ol>
<li><font color="#00aa00">git log</font> - просмотр логов, показывает дельту (разницу/diff), привнесенную каждым коммитом.
<pre>commit 9a452d9cdbdb57e7e4f2b09f8ce2f776cd56657a
Author: devlabuser &lt;user@mail.ru&gt;
Date:   Wed Jul 31 18:35:47 2013 +0400
    first commit

commit d528335724dfc15461996ed9d44d74f23ce6a075
Author: devlabuser &lt;user@mail.ru&gt;
Date:   Wed Jul 31 06:24:57 2013 -0700
    Initial commit
</pre>
</li>

<li>Копируем идентификатор коммита, до которого происходит откат.</li>

<li>Откатываемся до последнего успешного коммита (указываем последний коммит):
<pre>$ git reset --hard 9a452d955bdb57e7e4f2b09f8ce2fbb6cd56377a
HEAD is now at 9a45779 first commit
</pre>
Можно откатить до последней версии ветки:
<pre>$ git reset --hard origin/dev
HEAD is now at 9a45779 first commit
</pre>
</li>
</ol>

После того, как откат сделан, и выполнен очередной локальный коммит, при попытке сделать push в удаленный репозиторий, git может начать ругаться, что версия вашей ветки младше чем на github и вам надо сделать pull. Это лечится принудительным коммитом:
<pre>git push -f origin master</pre>

### Как выполнить слияние с другой веткой

<font color="#00aa00">git merge</font> выполняет слияние текущей и указанной ветки. Изменения добавляются в текущую ветку.
<pre>$ git merge origin/ticket_1001_branch</pre>

git pull забирает изменения из ветки на удаленном сервере и проводит слияние с активной веткой.
<pre>$ git pull origin ticket_1001_branch</pre>

<font color="#00aa00">git pull</font> отличается от <font color="#00aa00">git merge</font> тем, что <font color="#00aa00">merge</font> только выполняет слияние веток, а pull прежде чем выполнить слияние - закачивает изменения с удаленного сервера. <font color="#00aa00">merge</font> удобно использовать для слияния веток в локальном репозитории, <font color="#00aa00">pull</font> - слияния веток, когда одна из них лежит на github.

### Создание нового локального репозитория

<pre>$ mkdir project_dir
$ cd project_dir
$ git init

</pre>

### git cherry-pick

<font color="#00aa00">git cherry-pick</font> помогает применить один-единственный коммит из одной ветки к дереву другой.
<ol>
<li>Для этого нужно выписать ветку, в которую будем вливать коммит:
<pre>git checkout master</pre>
</li>

<li>Обновить ее:
<pre>git pull origin master</pre>
</li>

<li>Выполнить команду, указать код коммита:
<pre>git cherry-pick eb042098a5</pre>
</li>

<li>После этого обновить ветку на сервере:
<pre>git push origin master</pre>
</li>
</ol>

### Как раскрасить команды git

После создания репозитория в текущей директории появится субдиректория <font color="#00aa00">.git</font> . Она содержит файл <font color="#00aa00">config</font> .
<pre>[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[remote "origin"]
        fetch = +refs/heads/*:refs/remotes/origin/*
        url = git@github.com:devlab/sharp.git
[branch "master"]
        remote = origin
        merge = refs/heads/master
[branch "dev"]
        remote = origin
        merge = refs/heads/dev
</pre>
Чтобы раскрасить вывод git, можно добавить в файл блок <font color="#00aa00">[color]</font>:
<pre>[color]
        branch = auto
        diff = auto
        interactive = auto
        status = auto
        ui = auto
</pre>

