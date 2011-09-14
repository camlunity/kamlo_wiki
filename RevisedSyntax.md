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
`-pp camlp4r` в строку компиляции:
    $ ocamlfind ocamlopt -pp camlp4r mymodule.ml -o myexecutable

При компиляции через ocamlbuild добавьте в файл `_tags` тэг
`camlp4r` для файлов, написанных в revised syntax, например,
    <*.ml*> : camlp4r

Однако если добавлять тэг для всех `*.ml*`, а в проекте есть
.ml-файлы, сгенерированные ocamllex/ocamlyacc (в original syntax), нужно
будет исключить их:
    <mylexer.ml> | <myparser.ml{,i}> : -camlp4r, camlp4o

# Фразы

-   Точка с запятой завершает фразу. Двойная точка с запятой – не
    распознаётся. Последовательности операторов имеют другой синтаксис,
    и точка с запятой в конце фразы не будет перепутана с точкой с
    запятой, разделяющей последовательность операторов.
-   Объявление глобального значения производится ключевым словом
    `value`. Слово `let` зарезервировано
    исключительно для конструкции `let..in`:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          let x = 23;;

      </td>
          <td>

          let x = 23 in x + 7;;

      </td>
      </tr>
      <tr>
          <td>

          value x = 23;

      </td>
          <td>

          let x = 23 in x + 7;

      </td>
      </tr>
    </table>


-   В интерфейсах модулей вместо `val` следует использовать
    тоже `value`.

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          val x : int;;

       </td>
          <td>

          value x : int;

       </td>
      </tr>
    </table>

# Императивные конструкции

-   Появился ещё один способ определить последовательность операторов:
    за ключевым словом `do` следует последовательность
    операторов, разделённых `;`, окружённая `{` и
    `}` (допускается `;` после последнего
    оператора).

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          e1; e2; e3; e4

       </td>
          <td>

            do { e1; e2; e3; e4 }

       </td>
      </tr>
    </table>

-   Тело `for` и `while` может быть оформлено с
    тем же синтаксисом:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          while e1 do
          e2; e3; e4
          done

       </td>
          <td>

            while e1 do {
            e2; e3; e4
            }

       </td>
      </tr>
    </table>

# Туплы и списки

-   Скобки обязательны в туплах:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          1, "hello", World

       </td>
          <td>

            (1, "hello", World)`

       </td>
      </tr>
    </table>

-   Списки всегда окружены `[` и `]`. Их
    синтаксис:

    list      ::= [ elem-list opt-cons ]
    elem-list ::= expression ; elem-list | expression
    opt-cons  ::= :: expression | (*empty*)

-   Список – последовательность выражений, разделённых точкой с запятой,
    опционально завершённых `::` и выражением, и всё
    заключено в квадратные скобки. Например:


    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          x::y

       </td>
          <td>

          [x::y]

       </td>
      </tr>
      <tr>
          <td>

          [x; y; z]

       </td>
          <td>

          [x; y; z]

       </td>
      </tr>
      <tr>
          <td>

          x::y::z::t

       </td>
          <td>

          [x::[y::[z::t]]]

       </td>
      </tr>
      <tr>
          <td>

          x::y::z::t

       </td>
          <td>

          [x; y; z :: t]

       </td>
      </tr>
    </table>

-   Заметьте, есть два способа записать последний случай.

# Irrefutable patterns

Некоторые синтаксические конструкции используют понятие "irrefutable
patterns". Сопоставление значений таким паттернам всегда успешно. Вот
что может называться irrefutable pattern:

-   Переменная.
-   Символ `_`, соответствующий любому значению.
-   Конструктор тупла `()`.
-   Тупл с irrefutable patterns.
-   Запись с irrefutable patterns.
-   Irrefutable pattern с ограничением типа (type constraint).

Заметьте, что термин "irrefutable" применяется не ко всем паттернам,
сопоставление с которыми всегда удачно: например, тип с ровно одним
конструктором (кроме `() : unit`) не является irrefutable.

# Конструкции с паттерн матчингом

-   Ключевое слово `function` больше не существует,
    используйте `fun` вместо него.
-   При паттерн матчинге конструкций `fun`,
    `match` и `try` все варианты должны быть
    заключены в квадратные скобки: открывающаяся скобка `[`
    перед первым вариантом и закрывающаяся `]` после
    последнего:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          match e with
           p1 -> e
          | p2 -> e2;;

       </td>
          <td>

          match e with
          [ p1 -> e1
          | p2 -> e2 ];

       </td>
      </tr>
      <tr>
          <td>

          fun x -> x;;

       </td>
          <td>

          fun [x -> x];

       </td>
      </tr>
    </table>

-   Однако, если вариант только один и паттерн является irrefutable,
    квадратные скобки не обязательны. Следующие примеры работают
    одинаково в оригинальном и revised синтаксисах:


    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          fun x -> x

       </td>
          <td>

          fun x -> x

       </td>
      </tr>
      <tr>
          <td>

          fun {foo=(y, _)} -> y

       </td>
          <td>

          fun {foo=(y, _)} -> y

       </td>
      </tr>
    </table>

-   Каррированный паттерн матчинг может быть оформлен через
    `fun`, но только с irrefutable паттернами:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          fun x (y, z) -> t

       </td>
          <td>

          fun x (y, z) -> t

       </td>
      </tr>
      <tr>
          <td>

          fun x y (C z) -> t

       </td>
          <td>

          fun x y -> fun [C z -> t]

       </td>
      </tr>
    </table>

-   Теперь наконец-то стало возможно написать пустую функцию, кидающую
    исключение `Match_failure`, какой бы аргумент к ней ни
    применили, пустой `match`, кидающий
    `Match_failure` после вычисления значения выражения, и
    пустой `try`, эквивалентный выражению без
    `try`:

              fun []
              match e with []
              try e with []

-   Паттерны в `let` и `value` должны быть
    irrefutable. Следующее выражение, написанное в original syntax:

              let f (x::y) = ...

-   в revised syntax должно быть переписано так:

              let f = fun [ [x::y] -> ...

-   Конструкция `where` вернулась, но только с одной
    привязкой: можно написать `e where x = y`, но не
    `where x = y and z = t`

# Изменяемые значения и присваивание

-   Инструкция `<-` записывается как `:=`:


    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          x.f <- y

       </td>
          <td>

          x.f := y

       </td>
      </tr>
    </table>

-   Тип `ref` объявлен как запись с одним полем
    `val`, вместо `contents`. Оператор
    `!` более не существует, и присваивать значения ссылкам
    следует так же, как и любым другим изменяемым записям:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          x := !x + y

       </td>
          <td>

          x.val := x.val + y

       </td>
      </tr>
    </table>

# Типы

-   Конструкторы параметрических типов записываются перед параметрами, а
    параметры каррируются:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          int list

       </td>
          <td>

          list int

       </td>
      </tr>
      <tr>
          <td>

          ('a, bool) Hashtbl.t

       </td>
          <td>

          Hashtbl.t 'a bool

       </td>
      </tr>
      <tr>
          <td>

          type 'a foo =
            'a list list;;

       </td>
          <td>

          type foo 'a =
          list (list 'a);

       </td>
      </tr>
    </table>


-   В туплах из типов обязательны скобки:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          int * bool

       </td>
          <td>

          (int * bool)

       </td>
      </tr>
    </table>

-   В объявлении вариантного типа варианты должны быть заключены в
    квадратные скобки:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          type t = A of i | B;;

       </td>
          <td>

          type t = [ A of i | B ];

       </td>
      </tr>
    </table>

-   Возможно создать пустой (ненаселённый) тип, без конструктора:

              type foo = [];

-   Существует разница между конструкторами с несколькими параметрами и
    конструкторами с одним параметром, являющимся туплом. При объявлении
    конструктора с несколькими параметрами они разделены ключевым словом
    `and`. В выражениях и при паттерн-матчинге параметры
    конструктора каррируются:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          type t = C of t1 * t2;;

       </td>
          <td>

          type t = [ C of t1 and t2 ];

       </td>
      </tr>
      <tr>
          <td>

          C (x, y);;

       </td>
          <td>

          C x y;

       </td>
      </tr>
    </table>

-   Объявление конструктора с одним параметром-туплом осуществляется с
    использованием тупла типов. В выражениях и паттернах параметры не
    каррируются, а матчатся как обычный тупл:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          type t = D of (t1 * t2);;

       </td>
          <td>

          type t = [ D of (t1 * t2) ];

       </td>
      </tr>
      <tr>
          <td>

          D (x, y);;

       </td>
          <td>

          D (x, y);

       </td>
      </tr>
    </table>

-   Предопределённые конструкторы булевого типа `True` и
    `False` начинаются с заглавной буквы, как любые другие
    конструкторы.
-   В записях ключевое слово `mutable` должно быть после
    двоеточия:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          type t = {mutable x : t1};;

       </td>
          <td>

          type t = {x : mutable t1};

       </td>
      </tr>
    </table>

# Модули

- Применение модулей теперь каррируется:

   <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          type t = Set.Make(M).t;;

       </td>
          <td>

          type t = (Set.Make M).t;

       </td>
      </tr>
    </table>

# Полиморфные варианты

Объявляются с указанием на открытость/закрытость типа:

    type poly_open = [> `A | `B of (int * string) ];
    type poly_eq = [= `A |`B of (int * string) ];
    type poly_closed = [< `A | `B of (int * string) ];

# Объекты

Объекты объявляются с использованием `value` для полей:

    # object
        value my_field = 123;
        method my_method () = my_field;
      end;
    - : < my_method : unit -> int > = <obj>

# Разное

-   Часть `else` теперь обязательна в `if`:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          if a then b

       </td>
          <td>

          if a then b else ()

       </td>
      </tr>
    </table>

-   Булевые операторы `or` и `and` записываются
    только как `||` и `&&`:


    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          a or b & c

       </td>
          <td>

          a || b && c

       </td>
      </tr>
    </table>

# Streams and parsers

-   Streams и паттерны в stream parsers окружаются `[:` и
    `:]` вместо `[<` and `>]`.
-   Компонент stream'а записывается с обратным апострофом вместо
    обычного:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          [< '1; '2; s; '3 >]

       </td>
          <td>

          [: `1; `2; s; `3 :]

       </td>
      </tr>
    </table>

-   Варианты в парсере заключаются в `[` and `]`,
    как в `fun`, `match` и `try`. Если
    вариант только один, скобки не обязательны:

    <table border=1>
      <tr>
        <td>Ocaml</td>
        <td>Revised</td>
      </tr>
      <tr>
          <td>

          parser
          [< `Foo >] -> e
          | [< p = f >] -> f

       </td>
          <td>

          parser [ [: `x :] -> x ]

       </td>
      </tr>
      <tr>
          <td>

          parser [< 'x >] -> x

       </td>
          <td>

          parser [: `x :] -> x

       </td>
      </tr>

-   Возможно написать пустой парсер, кидающий исключение
    `Stream.Failure`, какой бы поток к нему ни применили.
    Пустой паттернг матчинг тоже кидает `Stream.Failure`:

              parser []
              match e with parser []

* * * * *

2011-03-26 13:08
