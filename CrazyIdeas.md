* * * * *

Свалка идей. Возможно фантастических. Возможно реальных.

## printf/scanf реализованные на camlp4

Цель - скорость, полностью избавиться от парсинга форматной строки в
рантайме.

См.
[http://caml.inria.fr/pub/ml-archives/caml-list/2002/04/1d6d46bf8aa8f5fa8ac3bdac54bf96d1.en.html](http://caml.inria.fr/pub/ml-archives/caml-list/2002/04/1d6d46bf8aa8f5fa8ac3bdac54bf96d1.en.html)

## generic print

При первом проходе - синтаксически заменять все `print expr` на
`let _x = expr in ()` и после компиляции смотреть тип expr через дамп
[TypedTree](kamlo_wiki/blob/master/TypedTree.md) или через -annot. При втором
проходе используя информацию о типах синтаксически собирать принтер
всего выражения из принтеров составных частей (которые определены для
примитивных типов и типов из stdlib) и подставлять этот конкретный
принтер в исходный код. Возможно задача сводится к автоматическому
выводу нужного аргумента (типа) для Show.show<\_\> (deriving) и
полуавтоматической реализации модулей Show\_ для существующих типов (из
stdlib).

Косяки :

-   полиморфные значение остаются полиморфными (т.е. не печатаются)
-   может влиять на вывод типов (?)

## Пофиксить thread pool в JoCaml

Есть подозрение что текущая реализация субоптимальна, из-за чего
создаётся много новых коротко-живущих потоков.

## async actions a la F\#

Чисто синтаксически преобразовывать
    let! sock = connect addr in
    let! ping = read sock in
    let answer = "pong " ^ ping in
    let! () = write sock answer in
    ()

в
    connect_async addr (fun sock ->
    read_async sock (fun ping ->
    let answer = "pong " & ping in
    write_async sock answer (fun () -> ())))

Все \*\_async вызовы обрабатываются через один общий асинхронный
менеджер.

Вопросы:

-   обработка ошибок
-   только для IO?
-   исключения
-   вообще возможно ли это только на уровне синтаксиса?
-   а не повторение [lwt](kamlo_wiki/blob/master/lwt.md) ли это?

## нотификация о перемещении для custom\_val

Дополнительный callback в структуре custom\_ops для нотификации о смене
адреса value при сборке мусора (два случая - minor-\>major и
compaction), позволит сишным биндингам явно сохранять указатель на ocaml
значения и обновлять его при (редком) перемещении.

см.
[http://chatlogs.jabber.ru/ocaml@conference.jabber.ru/2010/08/06.html\#18:54:04.836275](http://chatlogs.jabber.ru/ocaml@conference.jabber.ru/2010/08/06.html#18:54:04.836275)

* * * * *

2011-03-26 13:08
