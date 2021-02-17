# Ассемблер для начинающих. Введение

## Как установить MASM32</h2>
Скачать архив с сайта www.masm32.com, распаковать и запустить распакованный файл install.exe на выполнение. Он будет задавать очень много вопросов и требовать много подтверждений, но никаких специальных знаний для установки не потребуется. Я установила и скачала 11 версию masm.

В списке системных переменных нужно изменить значение переменной "PATH". Дописать в конце строки:
<pre>C:\masm32\bin</pre>

## Первая программа на ассемблере

Cтандартный "Hello World". Для вывода результатов я решила сразу использовать окна windows, а не консоль. По простой причине - так красивее.
<pre>.386
.model flat, stdcall
option casemap: none
 
include /masm32/include/windows.inc
include /masm32/include/user32.inc
include /masm32/include/kernel32.inc
 
includelib /masm32/lib/user32.lib
includelib /masm32/lib/kernel32.lib
 
.data
msg_title db "Title", 0
msg_message db "Hello world", 0
 
.code
start:
invoke MessageBox, 0, addr msg_message, addr msg_title, MB_OK
invoke ExitProcess, 0
end start
</pre>
Запускаем редактор C:/masm32/qeditor.exe . Вводим текст программы. Сохраняем. При сохранении обязательно надо указать вместе с именем файла расширение ".asm" . Выбираем пункт меню "Project" -&gt; "Build All". Если в коде нет ошибок, программа будет скомпилирована и рядом с файлом исходного кода появится симпатичный exe-файл.


## Вторая программа на ассемблере

Эта программа суммирует значение переменных и выводит результат.
<pre>.486
.model flat, stdcall
option casemap: none
 
include /masm32/include/windows.inc
include /masm32/include/user32.inc
include /masm32/include/kernel32.inc
 
includelib /masm32/lib/user32.lib
includelib /masm32/lib/kernel32.lib

include /masm32/macros/macros.asm 
uselib masm32, comctl32, ws2_32 

.data
msg_title db "Title", 0
A DB 1h
B DB 2h
buffer db 128 dup(?)
format db "%d",0

.code
start:

MOV AL, A
ADD AL, B

invoke wsprintf, addr buffer, addr format, eax
invoke MessageBox, 0, addr buffer, addr msg_title, MB_OK

invoke ExitProcess, 0

end start
</pre>

