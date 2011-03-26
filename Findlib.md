![image](logo.png)
-   Findlib

\
\
 [[LanguageSetup](LanguageSetup.html)] [[TitleIndex](TitleIndex.html)]
[[WordIndex](WordIndex.html)]

* * * * *

Findlib -
[http://projects.camlcity.org/projects/findlib.html/](http://projects.camlcity.org/projects/findlib.html/)
-- менеджер библиотек для Ocaml.

Библиотека, с точки зрения findlib, -- это набор файлов, реализующих
функционал библиотеки и файл META, определенного формата, в котором
записана информация о библиотеке: название, версия, описание,
зависимости, состав и др.

С помощью утилиты ocamlfind, входящей в состав findlib, можно
устанавливать, удалять и использовать во время компиляции библиотеки,
поддерживающие findlib.

Скрипт "topfind" предоставляет удобное средство работы с библиотеками из
топлевела.

-   = Использование в top-level = Комманды:
-   \#use "topfind" загружает findlib
-   \#list отображает список установленных в системе библиотек
-   \#require <имя библиотеки\> добавляет в load path необходимые для
    загрузки библиотеки пути и загружает библиотеку
-   \#camlp4o;; загружает camlp4 (стандартный синтаксис)
-   \#camlp4r;; загружает camlp4 (revised syntax)
-   Topfind.reset();; перезагружает библиотеки
-   \#thread;; включает нити(threads)

TODO: ocamlfind ocamlpc -package name1,name2

-   -linkpg написание META, установка библиотек

* * * * *

2011-03-26 13:09
