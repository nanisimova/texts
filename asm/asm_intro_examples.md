# Ассемблер для начинающих. Примеры простых программ

*Просто примеры работающих программ на masm32, без подробного разбора и т.п. Использование MessageBox, ExitProcess. Вызов WinAPI-функции с помощью invoke. Подключение файлов и библиотек с помощью includelib и include. Инструкции call, push, test, add, jz, jmp, jne, mov, inc, dec, cmp. Реализация цикла через .REPEAT и .UNTIL, .WHILE и .ENDW. Использование .IF, .ENDIF, .CONTINUE и .BREAK.*

## Программа 1. Получение данных из командной строки

Программа получает данные из командной строки и выводит их в небольшом windows-окне.
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

.code
start:

call GetCommandLine ; результат будет помещен в eax
 
push 0
push chr$("Command Line")
push eax ; текст для вывода берем из eax
push 0
call MessageBox

push 0
call ExitProcess

end start
</pre>
Код с вызовом функций можно было бы заменить кодом:
<pre>invoke GetCommandLine
invoke MessageBox, 0, eax, chr$("Command Line"), 0
invoke ExitProcess, 0
</pre>
<font color="#00aa00">invoke</font> - это встроенный макрос для упрощения кода, и при компиляции всё это преобразуется в ассемблерные команды.

Т.е. код
<pre>invoke MessageBox, 0, eax, chr$("Command Line"), 0</pre>

эквивалентен коду
<pre>push 0
push chr$("Command Line")
push eax
push 0
call MessageBox
</pre>
Стек - это удобное место для хранения информации. Чаще всего он используется при вызове функций.

Все WinAPI-функции созданы по соглашению <font color="#00aa00">stdcall</font>, то есть, передача аргументов в них производится через стек в обратном порядке. Возвращают эти функции значение в регистре eax.

## Программа 2. Сложение двух чисел

Программа складывает два числа, и проверяет результат. Если сумма равна 0 - выводится одно сообщение, если нет - другое.
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

.code
start:

mov eax, 123
mov ebx, -90 
add eax, ebx

test eax, eax 

jz zero 
invoke MessageBox, 0, chr$("В eax не 0!"), chr$("Info"), 0
jmp lexit

zero:
invoke MessageBox, 0, chr$("В eax 0!"), chr$("Info"), 0

lexit:
invoke ExitProcess, 0

end start
</pre>
В данной задаче самое интересное - это работа с метками, командами <font color="#00aa00">jz</font>, <font color="#00aa00">jmp</font> и <font color="#00aa00">test</font>.

<font color="#00aa00">test</font> - это операция логического сравнения двух операндов, размерностью байт, слово или двойное слово. В процессе выполняется операцию логического умножения: бит результата равен 1, если соответствующие биты операндов равны 1, в остальных случаях бит результата равен 0. Затем устанавливаются флаги, в том числе флаг ZF (zero flag), который равен 1, если результат логического умножения равен нулю.

Флаг ZF в дальнейшем используется для анализа результата.

<font color="#00aa00">jnz</font> - выполняет переход по указанной метке, если не установлен флаг ZF. Данная команда обычно используется с операциями сравнения, которые влияют на состояние флага ZF. Например, <font color="#00aa00">test</font> и <font color="#00aa00">cmp</font>.

<font color="#00aa00">jz</font> - выполняет переход по указанной метке, если установлен флаг ZF. Данная команда обычно используется с операциями сравнения, которые влияют на состояние флага ZF. Например, <font color="#00aa00">test</font> и <font color="#00aa00">cmp</font>.

<font color="#00aa00">jmp</font> - выполняет безусловный переход по указанной метке.

## Программа 3. Организация цикла

Программа использует repeat для организации цикла.
<pre>.data

msg_title db "Title", 0
A DB 1h
buffer db 128 dup(?)
format db "%d",0

.code
start:

mov AL, A
.REPEAT
inc AL

.UNTIL AL==7

invoke wsprintf, addr buffer, addr format, AL
invoke MessageBox, 0, addr buffer, addr msg_title, MB_OK
 
invoke ExitProcess, 0

end start
</pre>
<font color="#00aa00">inc</font> - это увеличение значения операнда в памяти или регистре на 1.
<pre>.data

msg_title db "Title", 0
buffer db 128 dup(?)
format db "%d",0

.code
start:

mov eax, 1
mov edx, 1

.WHILE edx==1
inc eax
.IF eax==7
.BREAK
.ENDIF
.ENDW 

invoke wsprintf, addr buffer, addr format, eax
invoke MessageBox, 0, addr buffer, addr msg_title, MB_OK
 
invoke ExitProcess, 0
</pre>
Директиву <font color="#00aa00">.BREAK</font> используется, чтобы прервать цикл и продолжить выполнение программы далее. Можно использовать директиву <font color="#00aa00">.CONTINUE</font> для прерывания выполнения кода внутри цикла и перехода к
очередной проверке условия в конструкции repeat и while.

## Программа 4. Сумма всех элементов массива

Программа суммирует значения всех элементов массива. Интересна работой с массивами и реализацией цикла типа "for".
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
x dd 0,1,2,3,4,5,6,7,8,9,10,11
n dd 12

buffer db 128 dup(?)
format db "%d",0

.code
start:
mov eax, 0
mov ecx, n
mov ebx, 0
L: add eax, x[ebx]
add ebx, type x
dec ecx
cmp ecx, 0
jne L

invoke wsprintf, addr buffer, addr format, eax
invoke MessageBox, 0, addr buffer, addr msg_title, MB_OK
 
invoke ExitProcess, 0

end start
</pre>

<font color="#00aa00">dec</font> - уменьшение значения операнда в памяти или регистре на 1.

<font color="#00aa00">cmp</font> - сравнение двух операндов, методом вычитания второго операнда из первого. По результатам вычисления устанавливаются флаги.

<font color="#00aa00">jne</font> выполняет переход по указанной метке, если результат сравнения операндов был отрицательным, и операнды НЕ равны друг другу.


