Getting Started
===============


Установка
---------

В репозиториях дистрибутива может не быть новейшей (сейчас это 3.12.1) версии компилятора.
Пользователи [Debian](http://debian.org) и [Ubuntu](http://ubuntu.com) могут получить
её через репозиторий [Debian OCaml Task Force](http://wiki.debian.org/Teams/OCamlTaskForce):

    $ apt-key adv --keyserver pgp.surfnet.nl --recv 49881AD3
    $ gpg -a --export 49881AD3 | sudo apt-key add -
    OK
    $ sudo tee -a /etc/apt/sources.list > /dev/null
    deb     http://ocaml.debian.net/debian/ocaml-3.12.1 sid main
    deb-src http://ocaml.debian.net/debian/ocaml-3.12.1 sid main
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
            Objective Caml version 3.12.1

    #

Топлевел окамла содержит ряд полезных [директив](http://caml.inria.fr/pub/docs/manual-ocaml/manual023.html#toc90)
-- команд, начинающихся с "#" (октоторп, решётка). Среди них есть
  
  * `#cd "dir";;` -- смена текущей директории на dir
  
  * `#use "file.ml";;` -- загрузка файла file.ml в топлевел построчно, как если бы его содержимое набиралось здесь
  
  * `#load "str.cma";;` -- загрузка скомпилированного в байткод файла str в топлевел
  
  * `#directory "dir";;` -- добавление директории dir в список путей, где искать скомпилированные файлы

При работе с внешними пакетам, раскиданными по файловой системе, писать #directory и #load 
весьма утомительно. Более того, также утомительна будет и компиляция&сборка твоих программ.
Поэтому на помощь приходит findlib (aka ocamlfind).

Findlib
-------

[Findlib](Findlib.md) -- средство управления пакетам для окамла. Он предоставляет ряд инструментов, помогающих
работать с библиотеками: загружать их, указывать зависимости и т.д. Также findlib добавляет директиву 
`#topfind` в топлевел. После выполнения этой директивый ты увидишь простую понятную справку:

    # #use "topfind";;
     - : unit = ()
     Findlib has been successfully loaded. Additional directives:
       #require "package";;      to load a package
       #list;;                   to list the available packages
       #camlp4o;;                to load camlp4 (standard syntax)
       #camlp4r;;                to load camlp4 (revised syntax)
       #predicates "p,q,...";;   to set these predicates
       Topfind.reset();;         to force that packages will be reloaded
       #thread;;                 to enable threads

То есть, для загрузки установленного пакета, вместо `#directory "/long/long/path/to/lib";;` 
и `#load "library.cma";;` (а то и нескольких файлов), можно писать `#require "library";;` -- 
которая добавит необходимые пути, загрузить необходимые зависимости и саму библиотеку. Ну не праздник ли?!


Hello world!
------------

    #!/usr/bin/env ocaml

    let () =
        print_endline "Hello world!";;

Дописать:

  * Компиляция в байткод и native code

Азы Оазиса
----------

Однако, окамл -- не скриптовый язык, и проводить всё время в репле не круто. Надо и программы писать,
а затем и собирать их! Для этого лучше всего воспользоваться [оазисом](http://oasis.forge.ocamlcore.org/index.php).
Оазис определяет единый формат метаинформации о пакете и единое средство установки -- `ocaml setup.ml -configure`, `ocaml setup.ml -build`, `ocaml setup.ml -install`. 
После его установки тебе должна быть доступна команда oasis

    oasis 
    E: No subcommand defined, call 'oasis -help' for help

Создай директорию для своего проекта. Рекомендуемая структура: `./src` -- директория для исходников, `./tests` -- директория для тестов. Самое время для оазиса, запусти `oasis quickstart` -- команду, которая создаст для тебя центральный в экосистеме оазиса файл _oasis. `quickstart` предложит ответить на несколько общих вопросов: название проекта, краткое описание, авторы и т.д. Последним будет вопрос про плагины:

  * DevFiles -- добавит привычные для unix-мира Makefile и configure -- которые будут вызывать команды оазиса
  * META -- поддержка ocamlfind, должен быть выбран, если вы пишите библиотеку, которая предполагает установку!
  * StdFiles -- сгенерирует файлы AUTHORS.txt, README.txt, INSTALL.txt -- полезно, особенно INSTALL.txt

Если ты ошибся -- ничего страшного, результирующий файл _oasis можно затем легко поправить руками. 
После общей информации можно создать запускаемый_файл/библиотику/документ и т.д. Начнём с исполняемого файла.

  * executable name -- имя результирующего исходника, скажем test
  * path -- путь, где лежат (или будут лежать) твои файлы (src)
  * MainIs -- файл, считающийся основным, точкой входа (main.ml)

На этом предлагаю остановиться (выбрать n) и записать полученный файл (w). В результате твой _oasis выглядит как-то так:

    OASISFormat: 0.2
    Name:        test
    Version:     0.0.1
    Synopsis:    test
    Authors:     Xavier
    License:     CC0
    Plugins:     DevFiles (0.2), META (0.2), StdFiles (0.2)

    Executable test
      Path:       src
      BuildTools: ocamlbuild
      MainIs:     main.ml

Команда `oasis setup` прочитает этот файл и создаст необходимую инфраструктуру для сборки и распространения проекта.
    
    I: File AUTHORS.txt doesn't exist, creating it.
    I: File INSTALL.txt doesn't exist, creating it.
    I: File Makefile doesn't exist, creating it.
    I: File README.txt doesn't exist, creating it.
    I: File _tags doesn't exist, creating it.
    I: File configure doesn't exist, creating it.
    I: File myocamlbuild.ml doesn't exist, creating it.
    I: File setup.ml doesn't exist, creating it.

Теперь командами `ocaml setup.ml -configure|-build` сожно конфигурировать/собирать проект.

Если ты пишешь библиотеку, то вместо секции Executable у тебя будет секция Library с примерно таким содержимым:

    Library libka
      Path:            src
      Modules:         Libka
      InternalModules: Util
      BuildDepends:    unix, camlp4
      NativeOpt:       -w A
      ByteOpt:         -w A

Поля BuildDepends, NativeOpt и ByteOpt ты можешь также использовать и при создании исполняемого файла. Они означают, соответственно: пакеты, от которых зависит твой проект; [опции компилятора](http://caml.inria.fr/pub/docs/manual-ocaml/manual025.html#toc100) в нативный код; [опции компилятора](http://caml.inria.fr/pub/docs/manual-ocaml/manual022.html#toc86) в байткод. Здесь мы включаем все предупреждения как ошибки.
