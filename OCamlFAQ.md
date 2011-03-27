* * * * *

# Англоязычные ресурсы

Существуют англоязычные списки часто задаваемых вопросов и ответов на
них:

-   [Objective CAML Tutorial](http://www.ocaml-tutorial.org/)
-   [зеркало
    ocaml-tutorial'а](http://mirror.ocamlcore.org/ocaml-tutorial.org/index.html)
-   [OCaml FAQ](http://caml.inria.fr/resources/doc/faq/index.en.html)
-   [OCaml experts'
    FAQ](http://caml.inria.fr/pub/old_caml_site/FAQ/FAQ_EXPERT-eng.html)
-   [Cocan's OCaml FAQ](http://www.cocan.org/cocan's_ocaml_faq)
-   [Caml quick reference
    guide](http://pauillac.inria.fr/cdrom_a_graver/www/caml/FAQ/qrg-eng.html)
-   [Writing efficient numerical code in Objective
    Caml](http://caml.inria.fr/pub/old_caml_site/ocaml/numerical.html)

# Книги

-   [Чьё-то введение в функциональное программирование на русском
    языке](http://gdsfh.dyndns.org/kamlo-ext/fp-lecs.pdf)
-   ["Introduction to Objective Caml" by Jason
    Hickey](http://www.cs.caltech.edu/courses/cs134/cs134b/book.pdf)
-   [Using, Understanding, and UnravelingThe OCaml
    Language](http://caml.inria.fr/pub/docs/u3-ocaml/)
-   [Developing Applications With Objective
    Caml](http://caml.inria.fr/pub/docs/oreilly-book/)
-   [Unix system programming in Objective Caml by Xavier Leroy and
    Didier Rémy](http://ocamlunix.forge.ocamlcore.org/)

# Коаны

-   [http://web.archive.org/web/20031105005932/www.bagley.org/\~doug/ocaml/Notes/okoans.shtml](http://web.archive.org/web/20031105005932/www.bagley.org/~doug/ocaml/Notes/okoans.shtml)
-   [http://eigenclass.org/hiki/fp-ocaml-koans](http://eigenclass.org/hiki/fp-ocaml-koans)

# Местное

\1. В чём различие между "=" и "=="?

-   "=" – структурное сравнение, "==" – физическое сравнение.

    Структурное сравнение – сравнение на равенство структуры значений.
    Если значение является массивом, его структура – массив определённой
    длины, состоящий из значений, поэтому при сравнении двух массивов на
    структурное равенство проверяется равенство длин массивов и
    структурное равенство каждого из их элементов. То есть, сравнение
    идёт рекурсивно по структуре значений. Поэтому оно может не
    завершаться в случаях циклических структур данных.

    Для изменяемых значений (массивов, ссылок, записей с изменяемыми
    полями) верно следующее: если изменение элемента/содержимого/поля
    `a1` влечёт за собой изменение `a2`, то `a1 == a2`.

    Подробнее – смотрите [описание модуля
    Pervasives](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Pervasives.html)
    про функции "=" и "==".

\2. Где точка входа в программу на окамле?

-   Одной конкретной точки входа нет. Программа состоит из модулей,
    линкуемых в определённом порядке. В этом же порядке, для каждого
    модуля, входящего в программу, вычисляются значения верхнего уровня
    модуля в порядке их определения (сверху вниз).
    Несмотря на это, рекомендуем сделать функцию наподобие `main`, из
    которой уже вызывать основные функции программы, и вычислять
    значение этой функции в качестве последнего вычисляемого значения
    верхнего уровня последнего линкуемого модуля. Остальные значения
    верхнего уровня рекомендуем делать такими, чтобы они не вызывали
    сайд-эффектов. То есть, что-то наподобие:
        let func1 a = ...
        let func2 b = ...
        let main () = (func1 123; func2 234)
        let () = main ()

\3. В чём различие между `let _ = ..` и `let () = ..`?

-   `let _ = ..` позволяет любое выражение справа – любое значение
    любого типа. `let () = ..` соответствует только значению
    `() : unit`. Если известно, что функция используется только для
    сайд-эффектов или, в общем случае, возвращает `()`{.backtick},
    безопаснее будет использовать `let () = func args`, чтобы случайно
    не пропустить частичное применение наподобие
        let _ = Printf.printf "two ints are: %i and %i" 123

    В этом случае не будет выведено ничего (к `_`{.backtick} будет
    привязана функция с типом `int -> unit`). `let () = ..` в этом
    случае сообщил бы, что тип этой функции не соответствует типу
    `unit`{.backtick}.

    Если же знаете, что функция возвращает значение известного типа (не
    `unit`{.backtick}), которое нужно игнорировать, можно улучшить
    ошибкозащищённость и поддерживаемость программы, указав, значение
    какого типа хотим игнорировать, например:
        let skip_line ch =
          let (_ : string) = input_line ch in ()

    Это полезно как для того, чтобы избежать случайного частичного
    применения как во время написания кода, так и на будущее: если в
    функцию будут добавлены параметры, будет то же самое частичное
    применение, не отлавливаемое компилятором в случае `let _ = ..`.

* * * * *

2011-03-26 13:09
