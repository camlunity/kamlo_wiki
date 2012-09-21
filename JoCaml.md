[JoCaml](http://jocaml.inria.fr) это диалект OCaml с встроенными в язык
примитивами [join calculus](http://moscova.inria.fr/join/index.shtml) --
модели описания конкурентных процессов. Начиная с версии 3.10 jocaml был
переписан с упором на бинарную совместимость с "родителем" (для этого
пришлось пожертвовать "перемещаемостью кода" между процессами). Зато
теперь лёгким движеним руки можно собирать любой проект jocaml'ем и
начинать экспериментировать.

Шаги:

-   замена компиляторов, ocamlopt -\> jocamlopt итп. (todo у ocamlbuild
    есть ключ -use-jocaml, проверить)
-   Новые ключевые слова - def reply и spawn (а также переопределённые
    or и &) - возможно придётся поправить код в некоторых местах чтобы
    избежать конфликтов.
-   При компиляции по умолчанию включены модули threads и unix, поэтому
    если они уже используются в проекте - будет конфликт с феерической
    ошибкой :
        Error: Files /usr/lib/ocaml/unix.cmxa and /usr/lib/ocaml/unix.cmxa
               both define a module named Unix

-   camlp4 не знает про новые синтаксические конструкции поэтому
    использовать jocaml вместе с синтаксическими расширениями не
    получится, но можно использовать camlp4 для файлов проекта которые
    не используют jocaml-синтаксис (благодаря бинарной совместимости). К
    сожалению простым способом "обучить" camlp4 синтаксису jocaml
    [невозможно](http://yquem.inria.fr/pipermail/jocaml-list/2009-October/000118.html).

FIXME пример myocamlbuild.ml

jocaml исопльзует рантайм ocaml поэтому все ограничения касательно
многопоточности остаются в силе. FIXME пример кода Асинхронные каналы
реализуются с помощью дополнительных потоков. FIXME как отображается
высокоуровневый код на системные потоки? - см.
[jocaml-list](http://yquem.inria.fr/pipermail/jocaml-list/2009-November/000125.html).
FIXME примеры использования (wide-finder, ray tracer, всё?)

См. также
---------

* [Официальный мануал](http://jocaml.inria.fr/doc/index.html)
* [Tutorial on join calculus and JoCaml](https://sites.google.com/site/winitzki/tutorial-on-join-calculus-and-its-implementation-in-ocaml-jocaml)
