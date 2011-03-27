* * * * *

# Как сгенерировать документацию, которую нигде не найдешь

Ниже описано, каким способом была сгенерирована документация к camlp4 и
ocamlbuild, доступная на сайте camlunity.ru. Предполагается, что в
каталоге ocaml-3.12 лежат исходники камля, и вы уже выполняли make
world.

## Документация к Camlp4

    cd ocaml-3.12.0
    cd _build/camlp4
    mkdir doc
    ocamldoc -d doc -html boot/Camlp4

## Документация к ocamlbuild

    cd ocaml-3.12.0
    cd _build/ocamlbuild
    mkdir doc
    ocamldoc -d doc -html *.ml

* * * * *

2011-03-26 13:09
