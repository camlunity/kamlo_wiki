![image](logo.png)
-   sqlgg

\
\
 [[LanguageSetup](LanguageSetup.html)] [[TitleIndex](TitleIndex.html)]
[[WordIndex](WordIndex.html)]

* * * * *

# sqlgg

Тут будет про [sqlgg](http://ygrek.org.ua/p/sqlgg) (до тех пор пока на
сайте не будет вменяемого туториала/документации). Что это такое и с чем
его едят?

## Что это такое?

Парсер sql запросов и генератор кода для связывания параметров и
результатов выполнения запроса с переменными целевого языка (включая
OCaml), с проверкой корректности типов во время компиляции. sqlgg это
**не ORM** (код генерируется на основе sql запросов, а не наоборот) и
**не валидатор sql** (хотя было бы неплохо). Типовыводилка пока что
очень примитивная.

## Пример использования

Собираем все запросы SQL, которые могут нам понадобиться в коде, в
отдельный файл, скажем, test.sql:

    CREATE TABLE IF NOT EXISTS test (word varchar, categ varchar, counter int);
    CREATE INDEX IF NOT EXISTS testidx ON test (word, categ);
    CREATE INDEX IF NOT EXISTS words_idx ON test (word);
    -- @test_word
    SELECT COUNT(*) FROM test WHERE word=@word AND categ=@categ;
    -- @update_by_word
    UPDATE test SET counter=counter+@counter WHERE word=@word AND categ=@categ;
    -- @insert_new
    INSERT INTO test (word, categ, counter) VALUES;
    -- @select_words
    SELECT word, counter FROM test WHERE categ=@categ LIMIT @limit;

Получаем камлевый код из test.sql (с помощью -name указываем имя
функтора) :
    $ sqlgg -gen ocaml -name Make test.sql > test_sql.ml

Код на OCaml, который будет использовать это:
    open Sqlite3

    module Sql = Test_sql.Make(Sqlgg_sqlite3) 
    (* Sqlgg_sqlite3 это один из модулей реализующий собственно код доступа к БД *)

    let () =
      let file = Sys.argv.(1) in
      let db = db_open file in

        (* имена этих функций выбраны автоматически *)
        Sql.create_test db;
        Sql.create_index_testidx db;
        Sql.create_index_words_idx db;

        (* NB именованные параметры *)
        Sql.insert_new db ~word:"make" ~categ:"verb" ~counter:3L;
        Sql.insert_new db ~word:"do" ~categ:"verb" ~counter:1L;
        Sql.insert_new db ~word:"man" ~categ:"noun" ~counter:1L;

        (* результат выборки COUNT - одна строка, поэтому тип функции int option, так же сработает и LIMIT 1 *)
        begin match Sql.test_word db ~word:"man" ~categ:"noun" with
        | Some _ -> print_endline "the man exists"
        | None -> print_endline "no man there"
        end;

        (* выборка из двух столбцов *)
        Sql.select_words db ~categ:"verb" ~limit:1L (Printf.printf "%s %Ld\n");

        ()

Копируем sqlgg\_traits.ml и sqlgg\_sqlite3.ml из sqlgg/impl и
компилируем окончательно:
    $ ocamlfind ocamlc -linkpkg -package sqlite3 sqlgg_traits.ml sqlgg_sqlite3.ml test_sql.ml test.ml -o test

Запускаем
    $ ./test test.db
    the man exists
    make 3

* * * * *

2011-03-26 13:09
