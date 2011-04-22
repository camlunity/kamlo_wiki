Getting Started
===============


Установка
---------

Несмотря на то, что версия языка -- 3.12 вышла около года назад, в стабильных релизах
[Debian](http://debian.org) и [Ubuntu](http://ubuntu.com) до сих пор доступна только
предыдущая версия -- 3.11; устранить это досадное недоразумении можно подключив
репозиторий [Debian OCaml Task Force](http://wiki.debian.org/Teams/OCamlTaskForce):

    $ gpg -a --export 49881AD3 | sudo apt-key add -
    OK
    $ sudo tee -a /etc/apt/sources.list > /dev/null
    deb     http://ocaml.debian.net/debian/ocaml-3.12.0 sid main
    deb-src http://ocaml.debian.net/debian/ocaml-3.12.0 sid main
    ^D
    $ sudo aptitude update
    ...

Теперь можно поставить минимально-необходимый набор пакетов:

    $ sudo aptitude install ocaml ocaml-findlib oasis3

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


Дописать:

  * topelevel directives и findlib
  * загрузка модулей


Hello world!
------------

    #!/usr/bin/env ocaml

    let () =
        print_endline "Hello world!";;

Дописать:

  * Компиляция в байткод и native code
  * Oasis
