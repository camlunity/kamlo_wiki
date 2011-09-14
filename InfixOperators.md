Тут мы собрали полезные инфиксные операторы.

    (** пропустить значение последовательно через функции:
        123 >> string_of_int >> print_string
    *)
    let ( >> ) x f = f x

test
> test2

    (** применить значение к функции:
        print_string & string_of_int & 123

        NB: оператор "&" является ключевым словом в jocaml

        Если попробовать объявить "let ( $ ) f x = f x",
        то полученный оператор будет левоассоциативным,
        что нежелательно в данном случае.
    *)
    let ( & ) f x = f x

    (** композиция функций:
        let print_int = print_string % string_of_int
        let print_int = print_string $ string_of_int
        let print_int_sum = print_string % string_of_int %% ( + )
        let print_int_sum = print_string %% (string_of_int %% ( + ) )
        let for_all pred = not % List.exists (not % pred)
        let for_all2 pred = not %% List.exists2 (not %% pred)

        Операторы левоассоциативны, у оператора ($) приоритет ниже,
        чем у (%), и ниже, чем у арифметических операторов.
    *)
    let ( % ) f g = fun x -> f (g x)
    let ( $ ) = ( % )
    let ( %% ) f g = fun x y -> f (g x y)
    let ( %%% ) f g = fun x y z -> f (g x y z)

    (** применить инфиксную функцию:
        123L /* Int64.add */ 234L
    *)
    let ( /* ) x y = y x
    let ( */ ) x y = x y


    (* Для удобного использования инфиксных операторов
       существует отличное решение: pa_do
       ( http://pa-do.forge.ocamlcore.org/ )
       Если использовать его не можете, то в качестве
       слабого подобия можно взять нижеследующие модули.
       Их названия имеют вид "Тип1_as_тип2", и при открытии
       такого модуля со значениями типа1 можно будет работать
       теми операторами, которыми обычно работают со значениями
       типа2.
       Например,
       let my_int64 =
         let module M =
           struct
             open Int32_as_int
             open Int64_as_float
             let x = (Int64.of_int32 (123l + 234l)) +. 345L
           end
         in
           M.x
    *)

    (* Замечание: для консистентности модули "Тип1_as_тип2"
       всегда должны переопределять одни и те же операторы.
    *)

    (* todo: добавить в Int* операции mod, rem, битовые *)

    module Int_as_int =
      struct
        let ( + ) = Pervasives.( + )
        let ( - ) = Pervasives.( - )
        let ( * ) = Pervasives.( * )
        let ( / ) = Pervasives.( / )
        let ( ~- ) = Pervasives.( ~- )
      end

    module Float_as_float =
      struct
        let ( +. ) = Pervasives.( +. )
        let ( -. ) = Pervasives.( -. )
        let ( *. ) = Pervasives.( *. )
        let ( /. ) = Pervasives.( /. )
        let ( ~-. ) = Pervasives.( ~-. )
      end


    (** TODO core, pa_do, pa_openin *)

    module Int32_as_int =
      struct
        let ( + ) = Int32.add
        let ( - ) = Int32.sub
        let ( * ) = Int32.mul
        let ( / ) = Int32.div
        let ( ~- ) = Int32.neg
      end

    module Int64_as_int =
      struct
        let ( + ) = Int64.add
        let ( - ) = Int64.sub
        let ( * ) = Int64.mul
        let ( / ) = Int64.div
        let ( ~- ) = Int64.neg
      end

    module Int_as_float =
      struct
        let ( +. ) = Pervasives.( + )
        let ( -. ) = Pervasives.( - )
        let ( *. ) = Pervasives.( * )
        let ( /. ) = Pervasives.( / )
        let ( ~-. ) = Pervasives.( ~- )
      end

    module Float_as_int =
      struct
        let ( + ) = Pervasives.( +. )
        let ( - ) = Pervasives.( -. )
        let ( * ) = Pervasives.( *. )
        let ( / ) = Pervasives.( /. )
        let ( ~- ) = Pervasives.( ~-. )
      end

    module Int32_as_float =
      struct
        let ( +. ) = Int32.add
        let ( -. ) = Int32.sub
        let ( *. ) = Int32.mul
        let ( /. ) = Int32.div
        let ( ~-. ) = Int32.neg
      end

    module Int64_as_float =
      struct
        let ( +. ) = Int64.add
        let ( -. ) = Int64.sub
        let ( *. ) = Int64.mul
        let ( /. ) = Int64.div
        let ( ~-. ) = Int64.neg
      end

    module Int_as_int_overflow =
      (* from http://alan.petitepomme.net/cwn/2004.06.22.html *)
      struct
        exception Overflow

        let ( + ) a b =
          let c = a + b in
          if (a lxor b) lor (a lxor (lnot c)) < 0 then c else raise Overflow

        let ( - ) a b =
          let c = a - b in
          if (a lxor (lnot b)) lor (b lxor c) < 0 then c else raise Overflow

        let ( * ) a b =
          let c = a * b in
          if Int64.of_int c = Int64.mul (Int64.of_int a) (Int64.of_int b)
          then c else raise Overflow

        let ( / ) a b =
          if a = min_int && b = -1 then raise Overflow else a / b

        let ( ~- ) x =
          if x <> min_int then -x else raise Overflow

      end

## See also

[http://alaska-kamtchatka.blogspot.com/2010/08/smooth-operators.html](http://alaska-kamtchatka.blogspot.com/2010/08/smooth-operators.html)

* * * * *

2011-03-26 13:09
