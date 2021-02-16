# Ошибка "Could not find any downloads that satisfy the requirement" в pip

**Исходные данные.** Операционная система Debian 8. С помощью apt-get установлен **pip** ($ apt-get install python-pip).
При просмотре списка пакетов, с опциями --uptodate и --outdated в выводе появляются строки **"Could not find any downloads that satisfy the requirement"**.
<font color="#00aa00"><pre>$ pip list --uptodate
Could not find any downloads that satisfy the requirement quodlibet
Could not find any downloads that satisfy the requirement utidylib
Could not find any downloads that satisfy the requirement reportbug
Could not find any downloads that satisfy the requirement python-musicbrainz2
Could not find any downloads that satisfy the requirement cddb
MySQL-python (1.2.5)
SOAPpy (0.12.22)
wsgiref (0.1.2)
</pre></font>

В моем случае, проблема заключалась в версии **pip** (нюансы работы старого **pip** с http и https).
Apt-get устанавливал очень старую версию:
<font color="#00aa00"><pre>$ pip -V
pip 1.5.6 from /usr/lib/python2.7/dist-packages (python 2.7)
</pre></font>
На все попытки сделать апдейты и апгрейды apt отвечал, что все ОК, версия 1.5 - это именно то, что мне требуется.

**Решение:**
<font color="#00aa00"><pre>$ pip install --upgrade pip
Downloading/unpacking pip from https://files.pythonhosted.org/packages/5f/25/
e52d3f31441505a5f3af41213346e5b6c221c9e086a166f3703d2ddaf940
/pip-18.0-py2.py3-none-any.whl#sha256=070e4bf493c7c2c9f6a08dd
797dd3c066d64074c38e9e8a0fb4e6541f266d96c
  Downloading pip-18.0-py2.py3-none-any.whl (1.3MB): 1.3MB downloaded
Installing collected packages: pip
  Found existing installation: pip 1.5.6
    Not uninstalling pip at /usr/lib/python2.7/dist-packages, owned by OS
Successfully installed pip
Cleaning up...
</pre></font>

Далее, принудительная перезагрузка **bash**:
<font color="#00aa00"><pre>$ exec bash</pre></font>

Проверяем версию **pip**:
<font color="#00aa00"><pre>$ pip -V
pip 18.0 from /usr/local/lib/python2.7/dist-packages/pip (python 2.7)
</pre></font>

Пробуем выполнить команду, которая приводила к выводу ошибки:
<font color="#00aa00"><pre>$ pip list --uptodate
Package      Version
MySQL-python 1.2.5
pip          18.0
six          1.11.0
SOAPpy       0.12.22
</pre></font>

