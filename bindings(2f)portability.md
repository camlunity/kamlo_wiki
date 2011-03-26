![image](logo.png)
-   bindings/portability

\
\
 [[LanguageSetup](LanguageSetup.html)] [[TitleIndex](TitleIndex.html)]
[[WordIndex](WordIndex.html)]

* * * * *

best practices как писать либу с [сишными
биндингами](camlunity.ru/bindings.html) чтобы она с минимумом проблем
собиралась на всех инкарнациях камля.

Scope: ocaml утилиты, сишный код, утилиты для
сборки/конфигурирования/установки NB: сейчас тут вперемешку проблемы,
решения, жалобы и нытьё

-   OCamlMakefile требует bash/sed т.е. для ocaml/msvc не очень годится
-   ocamlbuild требует bash (ссылка где брать)
-   хинт не забывать про расширения файлов (o/obj, a/lib etc)
-   msvc не понимает c99 - поэтому на gcc сразу давать -std=c89 чтобы
    отлавливать несоответствия, а также -Wall (и -Wextra -pedantic для
    полноты ощущений)
    -   комментарии //
    -   inline
    -   переменные только в начале блока

-   на какие define'ы можно ориентироваться (\_WIN32 - msvc, mingw,
    кстати на x64 что?, \_MSC\_VER - msvc only)
-   пути к заголовочным файлам и либам
    -   configure (как правильно делать, делать ли вообще, альтернативы)
    -   пример типичного косяка. Mysql Connector/C (aka libmysql):
        -   на linux заголовочные файлы в /usr/include/mysql/mysql.h
            т.е. в коде логично писать \#include <mysql/mysql.h\>
        -   в source же дистрибутиве (который будет использоваться на
            windows) под виндой путь будет например
            C:/mysql/include/mysql.h т.е. в коде надо писать \#include
            <header.h\> и передавать правильный путь в -I опцию для
            препроцессора.

    -   линковка, импорт либы под msvc не подходят для mingw, и
        наоборот. import либу по dll'ке можно сгенерировать руками

-   не стоит использовать -custom
    ([deprecated](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=256900#49),
    "загрязняет" все зависимые бинарники)
-   и прочие подобные косяки (перечислить. много их)

## Этапы сборки биндингов

-   компиляция и линковка сишного кода и получение stubs - статической
    (для native и custom byte сборок) и динамической (для byte)
    библиотек содержащих сишный код переходников и импорты требующихся
    внешних сишных функций
-   компиляция камлевой стороны (объявления external)
-   линковка камлевой стороны с вписыванием (-cclib) ссылок на stub
    библиотеки (в виде опций линкера) в тело cma/cmxa модулей (это
    необязательно, но очень удобно так как избавляет от неоходимости
    указывать имя stubs во время финальной линковки и стирает различие
    между обычной камлевой либой и биндингом с точки зрения процесса
    сборки)
-   линковка программы использующей биндинги. Тут надо предусмотреть
    случаи когда биндинг уже установлен или используется локально (опции
    -dllpath для ocamlrun и -I <локальный каталог\> для сишного линкера)

## Ocamlbuild

myocamlbuild.ml:
    open Ocamlbuild_plugin
    open Command
    open Printf


    (**
      [link_stubs name ?cclib stubs] embeds options into ocaml library [name] so that
      further linking with main program will find C [stubs] library
      @param name ocaml library name
      @param stubs C stubs module name 
      @param cclib extra -cclib options to pass to linker
    *)
    let link_stubs name ?(cclib=[]) stubs =
      let stubs_lib = sprintf "lib%s.%s" stubs !Options.ext_lib in
      let stubs_dll = sprintf "dll%s.%s" stubs !Options.ext_dll in
      let cclib = List.flatten & List.map (fun lib -> ["-cclib"; lib]) & ("-l"^stubs) :: cclib in
      List.iter (fun ext ->
        let file = sprintf "file:%s.%s" name ext in
        (* embed -l option into ocaml library so that C linker can find C stubs *)
        flag ["link"; "ocaml"; "library"; file] & atomize cclib;
        dep  ["link"; "ocaml"; "library"; file] [stubs_lib];
        if ext = "cma" then
        begin
          (* embed -l option into ocaml library so that ocamlrun can find dll stubs *)
          flag ["link"; "ocaml"; "library"; "byte"; file] & atomize ["-dllib"; "-l" ^ stubs;];
          dep  ["link"; "ocaml"; "library"; "byte"; file] [stubs_dll]
        end
      ) ["cma"; "cmxa"]


    let ocaml_lib_stubs name ?(dir="") ?(cclib=[]) stubs =
      link_stubs name ~cclib stubs;
      (* locally compiled programs (toplevel and tests) will use locally compiled library *)
      ocaml_lib name;
      (* find library locally (ocamlrun and C linker respectively) *)
      flag ["link"; "ocaml"; "byte"; "use_"^name] & atomize ["-dllpath"; !Options.build_dir / dir];
      flag ["link"; "ocaml"; "use_"^name] & atomize ["-I"; (if dir = "" then "." else dir);]


    ;;


    dispatch begin function
    | After_rules ->
      ocaml_lib_stubs "mylib" ~cclib:["-lotherlib"] "mylib_stubs";
      dep ["c"; "compile"; "file:mylib_stubs.c"] ["mylib_config.h"];
    | _ -> ()
    end

libmylib\_stubs.clib:
    mylib_extra.o
    mylib_stubs.o

Функции link\_stubs и ocaml\_lib\_stubs достаточно обобщённые и
портабельные (проверено - правильно работают на linux и ocaml/msvc). В
файле .clib перечисляются сишные модули из которых состоит сишная
сторона стабов, расширение всегда .o (да, это будет работать и с
ocaml/msvc, хотя cl и создаёт объектные файлы с расширением .obj).
Обратите внимание на имя файла - на такое соглашение (lib[stubs].clib)
завязаны встроенные правила ocamlbuild и это учитывается параметром
stubs функции link\_stubs). Если сишный код зависит от внешних библиотек
то эту зависимость надо указать явно :

-   для сишного компилятора указать пути к заголовочным файлам (тэг
    include\_otherlib\_c надо указать для соответствующих сишных файлов
    в \_tags) :
    `flag ["c"; "compile"; "include_otherlib_c"] & atomize ["-ccopt"; "-I" ^ path_to_otherlib_include]`
-   вшить -cclib зависимость от внешней библиотеки в результирующий
    cm{x,}a с помощью параметра `~cclib:["-lotherlib"]`

В примере зависимость mylib\_stubs.c от (локального) mylib\_config.h
приведёт к тому что последний будет скопирован предварительно в \_build
и его сможет найти сишный компилятор.

* * * * *

2011-03-26 13:09
