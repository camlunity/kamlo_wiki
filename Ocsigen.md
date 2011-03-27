* * * * *

# Ocsigen

Ocsigen - это исследовательский проект для разработки программных
технологий для веба. [http://www.ocsigen.org](http://www.ocsigen.org)

## Собираем Ocsigen из исходников

Сначала запасаемся свежими исходниками:
    darcs get http://ocsigen.org/darcs/lwt
    darcs get http://ocsigen.org/darcs/ocsigen
    darcs get http://ocsigen.org/darcs/obrowser
    darcs get http://ocsigen.org/datcs/ocsimore

Ставим, используя пакетные менеджеры или руками:

-   ocaml
-   findlib
-   pcre-ocaml
-   ocaml-ssl
-   ocamlnet
-   [ocaml-sqlite3](http://www.ocaml.info/home/ocaml_sources.html#toc13)
-   [ocamlduce](http://ocamlduce.forge.ocamlcore.org/)
-   [react](http://caml.inria.fr/cgi-bin/hump.en.cgi?contrib=682)
-   cryptokit
-   camlzip
-   [menhir](http://pauillac.inria.fr/~fpottier/menhir) для obrowser
-   pgocaml (для ocsimore)

Перед тем, как собирать lwt, надо собрать и установить react и open-ssl.
    $ cd react
    $ chmod +x build
    $ ./build
    $ INSTALLDDIR=/usr/local/lib/ocaml/site-lib/react ./build install

К сожалению, инсталлятор у этого пакета недобитый, придется руками
поправить файл META в каталоге, куда поставилась библиотека.

    $ cd lwt
    $ gmake
    $ gmake install

Ставим Ocamlduce. Для его сборки нужны сами исходники OCaml 3.11.1
Следуем указаниям в README в архиве ocamlduce-3.11.1.0.tar.gz и
компиляем, и ставим.

* * * * *

2011-03-26 13:09
