Camlp4
======


Тут -- информация про camlp4, синтаксический препроцессор, см. также
    [StreamParsers](kamlo_wiki/blob/master/camlp4-StreamParsers.md).

API [документация](http://camlunity.ru/docs/camlp4) для Camlp4 3.12 доступна на
сайте [camlunity](http://camlunity.ru).


Пример
------

```ocaml
$ ocaml
        Objective Caml version 3.11.1

# use "topfind";;
# #camlp4o;;
# open Camlp4.PreCast;;
# let expr = Gram.Entry.mk "expr";;
val expr : '_a Camlp4.PreCast.Gram.Entry.t = <abstr>
# EXTEND Gram
  expr: [ [ x = expr; "+"; y = expr -> x + y
          | e = INT -> int_of_string e ] ];
  END;;
- : unit = ()
# let _loc = Loc.ghost;;
val _loc : Camlp4.PreCast.Loc.t = <abstr>
# Gram.parse_string expr _loc "1+3";;
    - : int = 4
```


Quotations
----------

Quotations всегда пишите в
[revised синтаксисе](kamlo_wiki/blob/master/RevisedSyntax.md), иначе могут
вылезти трудноуловимые неочевидные косяки (или баги в самом Camlp4), основной
код расширения можно писать и в original. См. например
[PR\#5231](http://caml.inria.fr/mantis/view.php?id=5231)


Сборка
------

Пример командной строки для сборки расширения:

```bash
$ ocamlfind ocamlc -c -package camlp4.quotations.r,camlp4.extend -syntax camlp4o pa_xxx.ml
```

* При компиляции, препроцессору можно передать дополнительные опции:
  ``-pp опции`
* Используя ocamlfind, следует использовать `-ppopt` для передачи дополнительных
  ключей препроцессору

* Ключ `-no_quot` отключает обработку quotations, позволяя, например,
  использовать в коде ``<<``.

### OASIS (?)

Кто-нибудь, опишите необходимую магию?


Макросы
-------

DEFINE, IFDEF, etc

`ocamlfind -package camlp4.macro`


Отладка самого Camlp4
----------------------

Переменная окружения CAMLP4\_DEBUG

Но см. [PR\#5120](http://caml.inria.fr/mantis/view.php?id=5120)


Рецепты
-------

### Дамп кода в виде AST

```ocaml
$ echo 'let a = 2' > q.ml
$ camlp4o -filter Camlp4AstLifter q.ml
let loc = Loc.ghost
in
  Ast.StVal (loc, Ast.BFalse,
    Ast.BiEq (loc, Ast.PaId (loc, Ast.IdLid (loc, "a")),
      Ast.ExInt (loc, "2")))
```

### Печать AST

```ocaml
$ ocaml
        Objective Caml version 3.11.2

# #use "topfind";;
- : unit = ()
# #camlp4o;;
        Camlp4 Parsing version 3.11.2
# #require "camlp4.quotations";;
# open Camlp4.PreCast;; (* Будем использовать дефолтные модули Syntax Loc итд *)
# module P = Camlp4.Printers.OCaml.Make(Syntax);; (* Модуль для вывода в оригинальном синтаксисе *)
# let p = new P.printer ();; (* Создаём принтер *)
val p : P.printer = <obj>
# let _loc = Loc.ghost;; (* Пустой location для экспериментов *)
val _loc : Camlp4.PreCast.Loc.t = <abstr>
# let typ = <:ctyp< [ A | B ] >> in Format.printf "type is : %a\n%a\n" p#ctyp typ p#sig_item <:sig_item< type a = $typ$; >>;; (* содержимое quotation'ов в revised, а выводим в original! *)
type is : | A | B
type a = | A | B;;
- : unit = ()
```

Ресурсы
-------

* [Reading Camlp4](http://ambassadortothecomputers.blogspot.com/p/reading-camlp4.html)
