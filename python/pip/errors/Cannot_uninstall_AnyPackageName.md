# Ошибка "Cannot uninstall AnyPackageName. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall"

Ошибка возникает при попытке обновления установленного пакета:
<font color="#00aa00"><pre>$ pip install tornado --upgrade
Collecting tornado
  Downloading https://files.pythonhosted.org/packages/45/ec/f2a03a05
09bcfca336bef23a3dab0d07504893af34
fd13064059ba4a0503/tornado-5.1.tar.gz (516kB)
...

Successfully built tornado
Installing collected packages: futures, singledispatch, backports-abc, tornado
  Found existing installation: tornado 3.2.2
Cannot uninstall 'tornado'. It is a distutils installed project and thus we 
cannot accurately
 determine which files belong to it which would lead to only a partial uninstall.
</pre></font>

Решение, используем опцию "--ignore-installed":
<font color="#00aa00"><pre>$ pip install tornado --upgrade --ignore-installed
Collecting tornado
...
Installing collected packages: backports-abc, six, singledispatch, futures, 
tornado
Successfully installed backports-abc-0.5 futures-3.2.0 singledispatch-3.4.0.3 
six-1.11.0 tornado-5.1
</pre></font>

Проверяем результат, версия пакета изменилась и он находится в списке не требующих обновления:
<font color="#00aa00"><pre>$ pip list --uptodate
Package        Version
backports-abc  0.5
futures        3.2.0
MySQL-python   1.2.5
pip            18.0
singledispatch 3.4.0.3
six            1.11.0
SOAPpy         0.12.22
tornado        5.1
</pre></font>

