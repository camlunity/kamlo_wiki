![image](logo.png)
-   RevisedSyntax

\
\
 [[LanguageSetup](LanguageSetup.html)] [[TitleIndex](TitleIndex.html)]
[[WordIndex](WordIndex.html)]

* * * * *

Revised syntax – альтернатива классическому original syntax в OCaml. Он
проще, регулярнее, логичнее, и исправляет некоторые недостатки
оригинального синтаксиса, иногда вызывающие трудноуловимые баги. Этот
синтаксис был назван "righteous", но название не прижилось.

# Как использовать

Интерактивно:
    $ ocaml
            Objective Caml version 3.11.1

    # #load "dynlink.cma";;
    # #load "camlp4r.cma";;
            Camlp4 Parsing version 3.11.1

    # value test = do { (); 1 };
    value test : int = 1
    #

При компиляции вызовом ocamlc, ocamlopt или ocamlfind – добавьте
`-pp camlp4r`{.backtick} в строку компиляции:
    $ ocamlfind ocamlopt -pp camlp4r mymodule.ml -o myexecutable

При компиляции через ocamlbuild добавьте в файл `_tags`{.backtick} тэг
`camlp4r`{.backtick} для файлов, написанных в revised syntax, например,
    <*.ml*> : camlp4r

Однако если добавлять тэг для всех `*.ml*`{.backtick}, а в проекте есть
.ml-файлы, сгенерированные ocamllex/ocamlyacc (в original syntax), нужно
будет исключить их:
    <mylexer.ml> | <myparser.ml{,i}> : -camlp4r, camlp4o

# Фразы

-   Точка с запятой завершает фразу. Двойная точка с запятой – не
    распознаётся. Последовательности операторов имеют другой синтаксис,
    и точка с запятой в конце фразы не будет перепутана с точкой с
    запятой, разделяющей последовательность операторов.
-   Объявление глобального значения производится ключевым словом
    `value`{.backtick}. Слово `let`{.backtick} зарезервировано
    исключительно для конструкции `let..in`{.backtick}:

OCaml
Revised
`let x = 23;;`{.backtick}
`value x = 23;`{.backtick}
`let x = 23 in x + 7;;`{.backtick}
`let x = 23 in x + 7;`{.backtick}
-   В интерфейсах модулей вместо `val`{.backtick} следует использовать
    тоже `value`{.backtick}.

`OCaml`{.backtick}
`Revised`{.backtick}
`val x : int;;`{.backtick}
`value x : int;`{.backtick}

# Императивные конструкции

-   Появился ещё один способ определить последовательность операторов:
    за ключевым словом `do`{.backtick} следует последовательность
    операторов, разделённых `;`{.backtick}, окружённая `{`{.backtick} и
    `}`{.backtick} (допускается `;`{.backtick} после последнего
    оператора).

`OCaml`{.backtick}
`Revised`{.backtick}
`e1; e2; e3; e4`{.backtick}
`do { e1; e2; e3; e4 }`{.backtick}
-   Тело `for`{.backtick} и `while`{.backtick} может быть оформлено с
    тем же синтаксисом:

`OCaml`{.backtick}
`Revised`{.backtick}
`while e1 do`{.backtick} \
 `e2; e3; e4`{.backtick} \
 `done`{.backtick}
`while e1 do {`{.backtick} \
 `e2; e3; e4`{.backtick} \
 `}`{.backtick}

# Туплы и списки

-   Скобки обязательны в туплах:

`OCaml`{.backtick}
`Revised`{.backtick}
`1, "hello", World`{.backtick}
`(1, "hello", World)`{.backtick}
-   Списки всегда окружены `[`{.backtick} и `]`{.backtick}. Их
    синтаксис:

    list      ::= [ elem-list opt-cons ]
    elem-list ::= expression ; elem-list | expression
    opt-cons  ::= :: expression | (*empty*)

-   Список – последовательность выражений, разделённых точкой с запятой,
    опционально завершённых `::`{.backtick} и выражением, и всё
    заключено в квадратные скобки. Например:

`OCaml`{.backtick}
`Revised`{.backtick}
`x::y`{.backtick}
`[x::y]`{.backtick}
`[x; y; z]`{.backtick}
`[x; y; z]`{.backtick}
`x::y::z::t`{.backtick}
`[x::[y::[z::t]]]`{.backtick}
`x::y::z::t`{.backtick}
`[x; y; z :: t]`{.backtick}
-   Заметьте, есть два способа записать последний случай.

# Irrefutable patterns

Некоторые синтаксические конструкции используют понятие "irrefutable
patterns". Сопоставление значений таким паттернам всегда успешно. Вот
что может называться irrefutable pattern:

-   Переменная.
-   Символ `_`{.backtick}, соответствующий любому значению.
-   Конструктор тупла `()`{.backtick}.
-   Тупл с irrefutable patterns.
-   Запись с irrefutable patterns.
-   Irrefutable pattern с ограничением типа (type constraint).

Заметьте, что термин "irrefutable" применяется не ко всем паттернам,
сопоставление с которыми всегда удачно: например, тип с ровно одним
конструктором (кроме `() : unit`{.backtick}) не является irrefutable.

# Конструкции с паттерн матчингом

-   Ключевое слово `function`{.backtick} больше не существует,
    используйте `fun`{.backtick} вместо него.
-   При паттерн матчинге конструкций `fun`{.backtick},
    `match`{.backtick} и `try`{.backtick} все варианты должны быть
    заключены в квадратные скобки: открывающаяся скобка `[`{.backtick}
    перед первым вариантом и закрывающаяся `]`{.backtick} после
    последнего:

`OCaml`{.backtick}
`Revised`{.backtick}
`match e with`{.backtick} \
 `  p1 -> e1`{.backtick} \
 `| p2 -> e2;;`{.backtick}
`match e with`{.backtick} \
 `[ p1 -> e1`{.backtick} \
 `| p2 -> e2 ];`{.backtick}
`fun x -> x;;`{.backtick}
`fun [x -> x];`{.backtick}
-   Однако, если вариант только один и паттерн является irrefutable,
    квадратные скобки не обязательны. Следующие примеры работают
    одинаково в оригинальном и revised синтаксисах:

`OCaml`{.backtick}
`Revised`{.backtick}
`fun x -> x`{.backtick}
`fun x -> x`{.backtick}
`fun {foo=(y, _)} -> y`{.backtick}
`fun {foo=(y, _)} -> y`{.backtick}
-   Каррированный паттерн матчинг может быть оформлен через
    `fun`{.backtick}, но только с irrefutable паттернами:

`OCaml`{.backtick}
`Revised`{.backtick}
`fun x (y, z) -> t`{.backtick}
`fun x (y, z) -> t`{.backtick}
`fun x y (C z) -> t`{.backtick}
`fun x y -> fun [C z -> t]`{.backtick}
-   Теперь наконец-то стало возможно написать пустую функцию, кидающую
    исключение `Match_failure`{.backtick}, какой бы аргумент к ней ни
    применили, пустой `match`{.backtick}, кидающий
    `Match_failure`{.backtick} после вычисления значения выражения, и
    пустой `try`{.backtick}, эквивалентный выражению без
    `try`{.backtick}:

              fun []
              match e with []
              try e with []

-   Паттерны в `let`{.backtick} и `value`{.backtick} должны быть
    irrefutable. Следующее выражение, написанное в original syntax:

              let f (x::y) = ...

-   в revised syntax должно быть переписано так:

              let f = fun [ [x::y] -> ...

-   Конструкция `where`{.backtick} вернулась, но только с одной
    привязкой: можно написать `e where x = y`{.backtick}, но не
    `where x = y and z = t`{.backtick}

# Изменяемые значения и присваивание

-   Инструкция `<-`{.backtick} записывается как `:=`{.backtick}:

`OCaml`{.backtick}
`Revised`{.backtick}
`x.f <- y`{.backtick}
`x.f := y`{.backtick}
-   Тип `ref`{.backtick} объявлен как запись с одним полем
    `val`{.backtick}, вместо `contents`{.backtick}. Оператор
    `!`{.backtick} более не существует, и присваивать значения ссылкам
    следует так же, как и любым другим изменяемым записям:

`OCaml`{.backtick}
`Revised`{.backtick}
`x := !x + y`{.backtick}
`x.val := x.val + y`{.backtick}

# Типы

-   Конструкторы параметрических типов записываются перед параметрами, а
    параметры каррируются:

`OCaml`{.backtick}
`Revised`{.backtick}
`int list`{.backtick}
`list int`{.backtick}
`('a, bool) Hashtbl.t`{.backtick}
`Hashtbl.t 'a bool`{.backtick}
`type 'a foo =`{.backtick} \
 `  'a list list;;`{.backtick}
`type foo 'a =`{.backtick} \
 `  list (list 'a);`{.backtick}
-   В туплах из типов обязательны скобки:

`OCaml`{.backtick}
`Revised`{.backtick}
`int * bool`{.backtick}
`(int * bool)`{.backtick}
-   В объявлении вариантного типа варианты должны быть заключены в
    квадратные скобки:

`OCaml`{.backtick}
`Revised`{.backtick}
`type t = A of i | B;;`{.backtick}
`type t = [ A of i | B ];`{.backtick}
-   Возможно создать пустой (ненаселённый) тип, без конструктора:

              type foo = [];

-   Существует разница между конструкторами с несколькими параметрами и
    конструкторами с одним параметром, являющимся туплом. При объявлении
    конструктора с несколькими параметрами они разделены ключевым словом
    `and`{.backtick}. В выражениях и при паттерн-матчинге параметры
    конструктора каррируются:

`OCaml`{.backtick}
`Revised`{.backtick}
`type t = C of t1 * t2;;`{.backtick}
`type t = [ C of t1 and t2 ];`{.backtick}
`C (x, y);;`{.backtick}
`C x y;`{.backtick}
-   Объявление конструктора с одним параметром-туплом осуществляется с
    использованием тупла типов. В выражениях и паттернах параметры не
    каррируются, а матчатся как обычный тупл:

`OCaml`{.backtick}
`Revised`{.backtick}
`type t = D of (t1 * t2);;`{.backtick}
`type t = [ D of (t1 * t2) ];`{.backtick}
`D (x, y);;`{.backtick}
`D (x, y);`{.backtick}
-   Предопределённые конструкторы булевого типа `True`{.backtick} и
    `False`{.backtick} начинаются с заглавной буквы, как любые другие
    конструкторы.
-   В записях ключевое слово `mutable`{.backtick} должно быть после
    двоеточия:

`OCaml`{.backtick}
`Revised`{.backtick}
`type t = {mutable x : t1};;`{.backtick}
`type t = {x : mutable t1};`{.backtick}

# Модули

Применение модулей теперь каррируется:

`OCaml`{.backtick}
`Revised`{.backtick}
`type t = Set.Make(M).t;;`{.backtick}
`type t = (Set.Make M).t;`{.backtick}

# Полиморфные варианты

Объявляются с указанием на открытость/закрытость типа:
    type poly_open = [> `A | `B of (int * string) ];
    type poly_eq = [= `A |`B of (int * string) ];
    type poly_closed = [< `A | `B of (int * string) ];

# Объекты

Объекты объявляются с использованием `value`{.backtick} для полей:
    # object
        value my_field = 123;
        method my_method () = my_field;
      end;
    - : < my_method : unit -> int > = <obj>

# Разное

-   Часть `else`{.backtick} теперь обязательна в `if`{.backtick}:

`OCaml`{.backtick}
`Revised`{.backtick}
`if a then b`{.backtick}
`if a then b else ()`{.backtick}
-   Булевые операторы `or`{.backtick} и `and`{.backtick} записываются
    только как `||`{.backtick} и `&&`{.backtick}:

`OCaml`{.backtick}
`Revised`{.backtick}
`a or b & c`{.backtick}
`a || b && c`{.backtick}
`a || b && c`{.backtick}
`a || b && c`{.backtick}

# Streams and parsers

-   Streams и паттерны в stream parsers окружаются `[:`{.backtick} и
    `:]`{.backtick} вместо `[<`{.backtick} and `>]`{.backtick}.
-   Компонент stream'а записывается с обратным апострофом вместо
    обычного:

`OCaml`{.backtick}
`Revised`{.backtick}
`[< '1; '2; s; '3 >]`
`` [: `1; `2; s; `3 :] ``
-   Варианты в парсере заключаются в `[`{.backtick} and `]`{.backtick},
    как в `fun`{.backtick}, `match`{.backtick} и `try`{.backtick}. Если
    вариант только один, скобки не обязательны:

`OCaml`{.backtick}
`Revised`{.backtick}
`parser`{.backtick} \
 ``   [< `Foo >] -> e `` \
 `| [< p = f >] -> f`{.backtick}
`parser`{.backtick} \
 `` [ [: `Foo :] -> e `` \
 `| [: p = f :] -> f ]`{.backtick}
`parser [< 'x >] -> x`{.backtick}
`` parser [ [: `x :] -> x ] ``
`parser [< 'x >] -> x`{.backtick}
`` parser [: `x :] -> x ``
-   Возможно написать пустой парсер, кидающий исключение
    `Stream.Failure`{.backtick}, какой бы поток к нему ни применили.
    Пустой паттернг матчинг тоже кидает `Stream.Failure`{.backtick}:

              parser []
              match e with parser []

* * * * *

2011-03-26 13:08
