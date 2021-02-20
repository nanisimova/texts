# Модуль Image::Info. Получение META-данных изображения

Интересный модуль, который позволяет получить различные мета-данные об изображении.

Работает с форматами: <font color="#00aa00">BMP, GIF, ICO, JPEG, PNG, PPM, PGM, PBM, SVG, TIFF, XBM, XPM</font>. Следует учитывать то, что для каждого формата количество и тип возвращаемых данных могут отличаться.

Ниже - краткое описание и простой пример использования. Просто для того, чтобы иметь представление о существовании подобного модуля и принципе его работы.

## Краткое описание

Модуль позволяет использовать 5 функций: <font color="#00aa00">image_info, image_type, dim, html_dim, determine_file_type</font>.

### Функция image_info

Функция <font color="#00aa00">image_info()</font> является основной и самой востребованной.

Возвращает мета-информацию заданного изображения. Данные возвращаются в виде ссылки на хэш с предопределенными полями.

<b>Поля, общие для всех типов изображений</b>
<ul>
 	<li><font color="#00aa00">file_media_type</font> - MIME-тип файла. Содержит строку вида - "image/jpeg" или "image/png".</li>
 	<li><font color="#00aa00">file_ext</font> - расширение файла изображения. Содержит значение из трех символов, например: "png" или "jpg".</li>
 	<li><font color="#00aa00">width</font> - ширина изображения в пикселях.</li>
 	<li><font color="#00aa00">height</font> - высота изображения в пикселях.</li>
 	<li><font color="#00aa00">color_type</font> - содержит короткую строку с аббревиатурой цветовой модели, которая используется для данного изображения. Может принимать одно из значений:
<ul>
 	<li>Gray</li>
 	<li>GrayA</li>
 	<li>RGB</li>
 	<li>RGBA</li>
 	<li>CMYK</li>
 	<li>YCbCr</li>
 	<li>CIELab</li>
</ul>
Кроме того, для перечисленных значений может использоваться префикс "Indexed-" . Например, "Indexed-RGB".</li>
 	<li><font color="#00aa00">resolution</font> - предоставляет информацию о физическом размере графического документа. Единицей измерения служат <font color="#00aa00">dpi, dpm</font> или <font color="#00aa00">dpcm</font> ("dots per inch/cm/meter").</li>
</ul>
<b>Дополнительные поля</b>

Данные для этих полей предоставляются только в том случае, если исследуемое изображение содержит необходимую информацию:
<ul>
 	<li><font color="#00aa00">SamplesPerPixel</font></li>
 	<li><font color="#00aa00">BitsPerSample</font></li>
 	<li><font color="#00aa00">Comment</font></li>
 	<li><font color="#00aa00">Interlace</font></li>
 	<li><font color="#00aa00">Compression</font></li>
 	<li><font color="#00aa00">Gamma</font></li>
 	<li><font color="#00aa00">LastModificationTime</font></li>
</ul>
Подробное описание полей см. в документации модуля.

Пример использования функции <font color="#00aa00">image_info()</font>:

```perl
#!/usr/local/bin/perl

use Image::Info qw(image_info);

my $info = image_info("12.jpg");

print $info-&gt;{file_ext}; # вывод: jpg
print $info-&gt;{color_type}; # вывод: YCbCr
print $info-&gt;{resolution}; # вывод: 300 dpi
print $info-&gt;{SamplesPerPixel}; # вывод: 3
```

### Функция image_type

Функция возвращает ссылку на хэш с единственным ключом - "<font color="#00aa00">file_type</font>". Функция полезна, если требуется информация только о типе файла.

```perl
use Image::Info qw(image_type);

my $info = image_type("12.jg");
print $info-&gt;{file_type}; # вывод: JPEG
```

Как видно из примера, <font color="#00aa00">Image::Info</font> правильно опознал jpg-файл, даже с ошибкой в названии.

### Функция dim

Возвращает ширину и высоту изображения, берет их из хэша, полученного от <font color="#00aa00">image_info()</font>.

```perl
use Image::Info qw(image_info dim);

my $info = image_info("12.jpg");
my $dim = dim($info);

print $dim; # вывод: 2048x1536
```

Зачем это нужно было реализовывать отдельной функцией - не представляю...

### Функция html_dim

Функция делает то же, что и <font color="#00aa00">dim()</font>, но упаковывает данные в html-формат. После упаковки их можно смело подставлять в html-код.

```perl
use Image::Info qw(image_info html_dim);

my $info = image_info("12.jpg");
my $dim = html_dim($info);

print $dim; # переменная содержит в себе строку &lt; width = "2048" height = "1536" &gt;
```

### Функция determine_file_type

Определяет тип графического файла, принимая на вход его содержимое.
