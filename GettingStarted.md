Getting Started
===============


Установка
---------

Несмотря на то, что версия языка -- 3.12 вышла около года назад, в стабильных релизах
[Debian](http://debian.org) и [Ubuntu](http://ubuntu.com) до сих пор доступна только
предыдущая версия -- 3.11; устранить это досадное недоразумении можно подключив
репозиторий [Debian OCaml Task Force](http://wiki.debian.org/Teams/OCamlTaskForce):

    $ apt-key adv --keyserver pgp.surfnet.nl --recv 49881AD3
    $ gpg -a --export 49881AD3 | sudo apt-key add -
    OK
    $ sudo tee -a /etc/apt/sources.list > /dev/null
    deb     http://ocaml.debian.net/debian/ocaml-3.12.0 sid main
    deb-src http://ocaml.debian.net/debian/ocaml-3.12.0 sid main
    ^D
    $ sudo aptitude update
    ...

Теперь можно поставить минимально-необходимый набор пакетов:

    $ sudo aptitude install ocaml ocaml-findlib oasis

Дописать:

  * [Конфликт](http://www.openmirage.org/wiki/install) версий ``ncurses-*``
    пакетов в Ubuntu Maverick


REPL aka toplevel
-----------------

К сожалению, в отличие от подавляющего числа языков, имеющих интерактивную
консоль, OCaml не использует библиотеку
[readline](http://cnswww.cns.cwru.edu/php/chet/readline/readline.html/),
реализующую привычный интерфейс для работы со строками в консоли (история,
редактирование и т.д.). Поэтому рекомендуется запускать REPL вместе с
[rlwrap](http://utopia.knoware.nl/~hlub/rlwrap/):

    $ rlwrap ocaml
            Objective Caml version 3.12.0

    #

Топлевел окамла содержит ряд полезных [директив](http://caml.inria.fr/pub/docs/manual-ocaml/manual023.html#toc90)
-- команд, начинающихся с "#" (октоторп, решётка). Среди них есть
  * #cd "dir";; -- смена текущей директории на dir
  * #use "file.ml";; -- загрузка файла file.ml в топлевел построчно, как если бы его содержимое набиралось здесь
  * #load "str.cma";; -- загрузка скомпилированного в байткод файла str в топлевел
  * #direcotry "dir";; -- добавление директории dir в список путей, где искать скомпилированные файлы

При работе с внешними пакетам, раскиданными по файловой системе, писать #directory и #load 
весьма утомительно. Более того, также утомительна будет и компиляция&сборка твоих программ.
Поэтому на помощь приходит findlib (aka ocamlfind).

Findlib
-------




Hello world!
------------

    #!/usr/bin/env ocaml

    let () =
        print_endline "Hello world!";;

Дописать:

  * Компиляция в байткод и native code

Азы Оазиса
----------

Для того, чтобы 
