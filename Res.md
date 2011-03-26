![image](logo.png)
-   Res

\
\
 [[LanguageSetup](LanguageSetup.html)] [[TitleIndex](TitleIndex.html)]
[[WordIndex](WordIndex.html)]

* * * * *

Res -- так я называю аналог "error monad".

Лицензия -- "делайте что хотите", public domain.

res.mli:

    type res 'a 'e =
      [= `Ok of 'a
      |  `Error of 'e
      ]
    ;

    value return : 'a -> res 'a 'e
    ;

    value fail : 'e -> res 'a 'e
    ;

    value bind : ('a -> res 'b 'e) -> res 'a 'e -> res 'b 'e
    ;

    value ( >>= ) : res 'a 'e -> ('a -> res 'b 'e) -> res 'b 'e
    ;

    value catch : (unit -> res 'a 'e1) -> ('e1 -> res 'a 'e2) -> res 'a 'e2
    ;

    value wrap1 : ('a -> 'z)
               -> ('a -> res 'z exn)
    ;

    value wrap2 : ('a -> 'b -> 'z)
               -> ('a -> 'b -> res 'z exn)
    ;

    value wrap3 : ('a -> 'b -> 'c -> 'z)
               -> ('a -> 'b -> 'c -> res 'z exn)
    ;

    value wrap4 : ('a -> 'b -> 'c -> 'd -> 'z)
               -> ('a -> 'b -> 'c -> 'd -> res 'z exn)
    ;

    value wrap5 : ('a -> 'b -> 'c -> 'd -> 'e -> 'z)
               -> ('a -> 'b -> 'c -> 'd -> 'e -> res 'z exn)
    ;

    value res_exn : (unit -> 'a) -> res 'a exn
    ;

    value res_opterr : option 'e -> res unit 'e
    ;

    value res_optval : option 'r -> res 'r unit
    ;

    (* Ловит реальные исключения, доводя их "fail exn".
       Отличие от res_exn в том, что тут функция возвращает уже res.
       "res_exn f" = "catch_exn (return % f)"
       = "catch_exn (fun () -> return (f ()))"
     *)
    value catch_exn : (unit -> res 'a exn) -> res 'a exn
    ;

    (* Ловит как реальные исключения, так и res-ошибки (обязанные иметь
       тип exn), обрабатывает. *)
    value catch_all : (unit -> res 'a exn) -> (exn -> res 'a exn) -> res 'a exn
    ;

    value exn_res : res 'a exn -> 'a
    ;

    value map_err : ('e1 -> 'e2) -> res 'a 'e1 -> res 'a 'e2
    ;

    value foldres_of_fold :
            ( ('a -> 'i ->     'a   ) -> 'a -> 'v ->     'a    ) ->
            ( ('a -> 'i -> res 'a 'e) -> 'a -> 'v -> res 'a 'e )
    ;

    (*
    value rprintf : Pervasives.format 'a Pervasives.out_channel unit -> res 'a exn
    ;
    *)

    value rprintf :
      Pervasives.format4
      'a unit string (res unit exn) -> 'a
    ;

    value reprintf :
      Pervasives.format4
      'a unit string (res unit exn) -> 'a
    ;

    value wrap_with1 :
       ( 'a -> ('r ->     'z    ) ->     'z     ) ->
       ( 'a -> ('r -> res 'z exn) -> res 'z exn )
    ;

    value wrap_with3 :
       ( 'a -> 'b -> 'c -> ('r ->     'z    ) ->     'z     ) ->
       ( 'a -> 'b -> 'c -> ('r -> res 'z exn) -> res 'z exn )
    ;


    (* sequential. *)
    value list_map_all :
       ( 'a -> res 'b 'e ) -> list 'a -> res (list 'b) ('a * 'e)
    ;

    value array_map_all :
       ( 'a -> res 'b 'e ) -> array 'a -> res (array 'b) ('a * 'e)
    ;


    value list_fold_left_all :
       ( 'a -> 'i -> res 'a 'e ) -> 'a -> list 'i
       -> res 'a ('i * list 'i * 'a * 'e)
    ;


    value list_iter_all :
       ( 'a -> res unit 'e ) -> list 'a -> res unit ('a * list 'a * 'e)
    ;


    (* error contains: (the_occured_error, count_of_repeats_made) *)
    value repeat : int -> ( 'a -> res 'a 'e ) -> 'a -> res 'a ('e * int)
    ;

res.ml:

    type res 'a 'e =
      [= `Ok of 'a
      |  `Error of 'e
      ]
    ;

    value return r = `Ok r
    ;

    value fail e = `Error e
    ;

    value bind f m =
      match m with
      [ `Ok a -> f a
      | (`Error _) as e -> e
      ]
    ;

    value ( >>= ) m f = bind f m
    ;


    value catch func handler =
      match func () with
      [ (`Ok _) as r -> r
      | `Error e -> handler e
      ]
    ;


    value wrap1 f = fun a ->
      try `Ok (f a)
      with [ e -> `Error e ]
    ;

    value wrap2 f = fun a b ->
      try `Ok (f a b)
      with [ e -> `Error e ]
    ;

    value wrap3 f = fun a b c ->
      try `Ok (f a b c)
      with [ e -> `Error e ]
    ;

    value wrap4 f = fun a b c d ->
      try `Ok (f a b c d)
      with [ e -> `Error e ]
    ;

    value wrap5 f = fun a b c d e ->
      try `Ok (f a b c d e)
      with [ e' -> `Error e' ]
    ;

    value catch_exn func =
      try
        func ()
      with
      [ e -> fail e ]
    ;

    value catch_all f handler =
      catch (fun () -> catch_exn f) handler
    ;

    value exn_res r =
      match r with
      [ `Ok x -> x
      | `Error e -> raise e
      ]
    ;

    value map_err f r =
      match r with
      [ (`Ok _) as r -> r
      | `Error e -> `Error (f e)
      ]
    ;


    value res_opterr oe =
      match oe with
      [ None -> `Ok ()
      | Some e -> `Error e
      ]
    ;


    value res_optval ov =
      match ov with
      [ None -> `Error ()
      | Some v -> `Ok v
      ]
    ;


    value ( & ) f x = f x
    ;


    value ( % ) f g = fun x -> f (g x)
    ;


    value res_exn func =
      catch_exn (return % func)
    ;


    exception Foldres_exit
    ;

    value (foldres_of_fold :
             ( ('a -> 'i ->     'a   ) -> 'a -> 'v ->     'a    ) ->
             ( ('a -> 'i -> res 'a 'e) -> 'a -> 'v -> res 'a 'e )
          )
    fold =
      fun f init v ->
        let opt_err = ref None in
        let new_f a v =
          match f a v with
          [ `Ok new_a -> new_a
          | `Error e -> (opt_err.val := Some e; raise Foldres_exit)
          ]
        in
        try
          `Ok (fold new_f init v)
        with
        [ Foldres_exit ->
            match opt_err.val with
            [ None -> assert False
            | Some e -> `Error e
            ]
        ]
    ;


    value rprintf fmt =
      Printf.ksprintf
        (fun str ->
           try
             return & output_string stdout str
           with
           [ e -> `Error e ]
        )
        fmt
    ;


    value reprintf fmt =
      Printf.ksprintf
        (fun str ->
           try
             return & (output_string stderr str; flush stderr)
           with
           [ e -> `Error e ]
        )
        fmt
    ;


    value wrap_with1 =
      fun with1 ->
        fun a f ->
          res_exn & fun () ->
            with1 a (exn_res % f)
    ;


    value wrap_with3 =
      fun with3 ->
        fun a b c f ->
          res_exn & fun () ->
            with3 a b c (exn_res % f)
    ;


    value list_map_all func lst =
      inner [] lst
      where rec inner rev_acc lst =
        match lst with
        [ [] -> return & List.rev rev_acc
        | [h :: t] ->
            match func h with
            [ `Ok x -> inner [x :: rev_acc] t
            | `Error e -> `Error (h, e)
            ]
        ]
    ;


    value array_map_all func arr =
      let lst = Array.to_list arr in
      list_map_all func lst >>= fun res_lst ->
      return & Array.of_list res_lst
    ;


    value list_fold_left_all func init lst =
      inner init lst
      where rec inner init lst =
        match lst with
        [ [] -> return init
        | [h :: t] ->
            match func init h with
            [ `Ok x -> inner x t
            | `Error e -> `Error (h, t, init, e)
            ]
        ]
    ;


    value list_iter_all func lst =
      catch
        (fun () ->
      list_fold_left_all
        (fun () x -> ((func x) : res unit _))
        ()
        lst
        )
        (fun (h, t, (), e) -> fail (h, t, e))
    ;


    value repeat n f a =
      inner 0 a
      where rec inner made a =
        if made >= n
        then
          `Ok a
        else
          match f a with
          [ `Ok a -> inner (made + 1) a
          | `Error e -> `Error (e, made)
          ]
    ;

* * * * *

2011-03-26 13:08
