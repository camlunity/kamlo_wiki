* * * * *

todo: описать, как обстоят дела с потоками (тредами) в окамле.

Advanced:

see also:
[http://snipplr.com/view/19509/ocaml-memory-allocation-and-context-switching/](http://snipplr.com/view/19509/ocaml-memory-allocation-and-context-switching/)
[http://snipplr.com/view/19485/ocaml-threads-context-switch-example/](http://snipplr.com/view/19485/ocaml-threads-context-switch-example/)

# in/out\_channel & threads

Стандартный ввод/вывод с помощью
[in\_channel/out\_channel](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Pervasives.html#TYPEin_channel).
Буферизированный, дополнительный IO lock при любых операциях с
in/out\_channel, дорого обходится в случае многопоточного io, позволяет
в простых случаях не задумываясь писать в stdout из разных потоков.
Альтернативы:

-   Unix.read/write (с буферизацией)
-   memory mapped файлы (см.
    [bigarray](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Bigarray.Genarray.html#VALmap_file))
-   асинхронное IO
    ([libaio-ocaml](http://libaio-ocaml.forge.ocamlcore.org/) для linux)

См.
[http://old.nabble.com/Threads-performance-issue.-td22039100.html](http://old.nabble.com/Threads-performance-issue.-td22039100.html)

# сишные биндинги, enter\_blocking\_section, gc

Рассмотрим [сишный биндинг](kamlo_wiki/blob/master/bindings.md) к гипотетической
функции `void write(char *, size_t)`, которая синхронно пишет данные в
сокет и может заблокироваться (ожидать IO на сокете, поток "засыпает"
внутри функции ОС).

### Вариант 1 (рабочий)

    CAMLextern value caml_write(value str)
    {
      CAMLparam1(str); // регистрируем параметр (для gc)
      write(String_val(str), caml_string_length(str));
      CAMLreturn(Val_unit); // разрегистрируем ранее зарегистрированные локальные value этой функции
    }

**Проблема:** если функция write блокируется, то блокируется и весь
поток, а из-за global runtime lock блокируется и весь процесс. Попробуем
это исправить, перед вызовом write отпустим рантайм лок, а после вызова
возьмём обратно. Таким образом другие камлевские потоки смогут
выполняться даже если write заблокируется. Очевидно эта оптимизация
имеет смысл только для многопоточных приложений.

### Вариант 2 (бажный)

    CAMLextern value caml_write(value str)
    {
      CAMLparam1(str); // регистрируем параметр (для gc)
      caml_enter_blocking_section(); // отпускаем рантайм лок, "входим в блокирующую секцию кода"
      write(String_val(str), caml_string_length(str));
      caml_leave_blocking_section(); // берём рантайм лок, "выходим из блокирующей секции кода"
      CAMLreturn(Val_unit); // разрегистрируем ранее зарегистрированные локальные value этой функции
    }

**Пробема:** в многопоточном приложении после того как рантайм лок
отпущен (после caml\_enter\_blocking\_section), им может завладеть
другой поток. И в какой-то момент может вызваться цикл [сборки
мусора](kamlo_wiki/blob/master/Gc.md), и в результате перемещения блоков памяти
(уплотнения) буфер занимаемый `str` может тоже переместиться. В
зависимости от разных факторов это может произойти до вызова `write` или
во время. В первом случае последующий вызов `caml_string_length`
прочитает мусор на месте `str` и в лучшем случае закрэшится (а в худшем
передаст неверное значение в `write`). Во втором случае `write`
прочитает часть настоящих данных, а часть - произвольный мусор. Обратите
внимание что это случайный процесс - может закрэшиться, может записать
мусор, а может и отработать корректно (в большинстве случаев). Поэтому
баги в многопоточных приложениях находить очень весело.

### Вариант 3 (рабочий)

    CAMLextern value caml_write(value str)
    {
      CAMLparam1(str); // регистрируем параметр (для gc)
      size_t len = caml_string_length(str);
      char* buf = (char*)caml_stat_alloc(len); // выделяем временный буфер (вне камлевского хипа), malloc + проверка на NULL
      memcpy(buf,String_val(str), len); // копируем во временный буфер
      caml_enter_blocking_section(); // отпускаем рантайм лок, "входим в блокирующую секцию кода"
      write(buf, len); // используем временный буфер
      caml_stat_free(buf); // чистим за собой (можно внутри blocking_section)
      caml_leave_blocking_section(); // берём рантайм лок, "выходим из блокирующей секции кода"
      CAMLreturn(Val_unit); // разрегистрируем ранее зарегистрированные локальные value этой функции
    }

Это стандартный, годный код для таких биндингов. Ценой лишнего
malloc+копирования избегается блокирование всего процесса.

**Золотое правило:** Ни в коем случае никаким образом не трогать (читать
или писать) блоки памяти в камлевском хипе внутри
enter/leave\_blocking\_section.

TODO: другие более advanced (менее удобные, более сишные) варианты если
профайлинг говорит что копировать буфер слишком дорого.

* * * * *

2011-03-26 13:08
