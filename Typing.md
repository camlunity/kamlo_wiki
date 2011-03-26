![image](logo.png)
-   Typing

\
\
 [[LanguageSetup](LanguageSetup.html)] [[TitleIndex](TitleIndex.html)]
[[WordIndex](WordIndex.html)]

* * * * *

Здесь будут описаны некоторые интересные вещи, касающиеся типизации.

Пока кучей, неотсортированные.

# let-полиморфизм

Источник -- "Types and Programming Languages" by B.C.Pierce. В общем
случае
    let a = b in c

не эквивалентно
    (fun a -> c) b

с точки зрения типизации. Эта неэквивалентность называется
let-полиморфизм.

    $ ocaml
            Objective Caml version 3.10.2

    # let () = let d x = ignore x in (d 0; d "");;
    # let () = (fun d -> (d 0; d "")) ignore;;
    This expression has type string but is here used with type int
    # 

Ошибка возникает из-за того, что при унификации лямбда-аргумента (`d`) с
выражениями `d 0` и `d ""` ему присваивается один фиксированный тип
(либо `int -> 'a`, либо `string -> 'a`), тогда как при типизации
выражения `let a = b in c` сначала вычисляется наиболее общий тип
выражения `b`, и только потом типизируется `c`, учитывая, в том числе,
тип значения `a`.

# rank-2 полиморфизм

Следующий код не проходит проверку типов:
    type t = MyInt of int | MyFloat of float | MyString of string
    let foo printerf = function
      | MyInt i -> printerf string_of_int i
      | MyFloat x -> printerf string_of_float x
      | MyString s -> printerf (fun x -> x) s

потому что первый оператор вызывает вычисление типа аргумента
`printerf`, и компилятор вычисляет тип `(int -> string) -> int -> unit`
вместо более полезного здесь типа `('t -> string) -> 't -> unit`. Для
того, чтобы проверка типов не выводила конкретный ограниченный тип,
можно завернуть полиморфную функцию в запись или объект с полиморфным
полем/методом, тогда выведенный тип будет типом записи/объекта, а его
поле сможем использовать с разными конкретными типами.
    # type t = MyInt of int | MyFloat of float | MyString of string

    type pr = { printerf : 'a . ('a -> string) -> 'a -> unit }

    let foo {printerf=printerf} = function
      | MyInt i -> printerf string_of_int i
      | MyFloat x -> printerf string_of_float x
      | MyString s -> printerf (fun x -> x) s

    let my_pr = { printerf = fun f x -> Printf.printf "res = %s\n%!" (f x) }

    let () = foo my_pr (MyInt 123)
    let () = foo my_pr (MyString "qwe")
    ;;

    res = 123
    res = qwe
    type t = MyInt of int | MyFloat of float | MyString of string
    type pr = { printerf : 'a. ('a -> string) -> 'a -> unit; }
    val foo : pr -> t -> unit = <fun>
    val my_pr : pr = {printerf = <fun>}
    # 

Запись вида ` { printerf :`**` 'a .`**` ('a -> string) -> 'a -> unit } `
означает, что **для любого типа `'a`** поле `printerf` имеет тип
`('a -> string) -> 'a -> unit`. Тип аргумента, принимаемого функцией
`foo`, тоже вычисляется, но теперь этот тип равен `pr`, то есть, тип не
полиморфный.

# Value restriction

Нижеследующий код страдает от value restriction:
    # let ( >> ) f g x = g (f x);;
    val ( >> ) : ('a -> 'b) -> ('b -> 'c) -> 'a -> 'c = <fun>
    # let is_zero n = (n = 0);;
    val is_zero : int -> bool = <fun>
    # List.length >> is_zero;;
    - : '_a list -> bool = <fun>

Тип композиции выводится не полиморфный. Объяснение -- ["Introduction to
Objective Caml" by Jason
Hickey](http://www.cs.caltech.edu/courses/cs134/cs134b/book.pdf), глава
5.1.1.

# Obj.magic

Функция с типом `'a -> 'b` - отличный способ отстрелить себе ногу, если
сильно хочется.

[http://caml.inria.fr/pub/ml-archives/caml-list/2007/04/200afc54d9796dbb7a8d75bef70f2496.en.html](http://caml.inria.fr/pub/ml-archives/caml-list/2007/04/200afc54d9796dbb7a8d75bef70f2496.en.html)

* * * * *

2011-03-26 13:08
