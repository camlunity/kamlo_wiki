![image](logo.png)
-   bindings/tips

\
\
 [[LanguageSetup](LanguageSetup.html)] [[TitleIndex](TitleIndex.html)]
[[WordIndex](WordIndex.html)]

* * * * *

Вспомогательные хитрости для описания биндингов к C функциям

Если сишная функция имеет в аргументах и возвращаемом результате базовые
типы, то достаточно описать только external в .ml (пример неудачный):
    value add(value a, value b) {
       return Val_int(Int_val(a) + Int_val(b));
    }

    external add : int -> int -> int = "add" "noalloc"

Здесь флагом “noalloc” мы помечаем, что функция не нуждается в работе с
хипом OCaml (не будет резервировать память в хипе или модифицировать
что-либо в хипе или отпускать runtime лок) и на этом можно сэкономить
работу с gc (не занимайтесь преждевременной оптимизацией если не уверены
- noalloc можно и имеет смысл использовать только в очень ограниченном
числе случаев).

Если предполагается, что мы биндим функцию 1:1, то есть внутри биндинга
данной функции не требуется проводить какой-либо специальной обработки,
то лучше всего применять нижеследующие дефайны, поскольку это позволяет
избегать большинство опечаток.

    #define Unit(x) ((x), Val_unit)

    #define ML_1(name, conv, retval)                                        \
      CAMLprim value ml_##name(value v) {return retval (name(conv(v)));}

    #define ML_2(name, conv1, conv2, retval)              \
      CAMLprim value ml_##name(value v1, value v2) { \
        return retval (name(conv1(v1), conv2(v2))); }

    #define ML_3(name, conv1, conv2, conv3, retval)                 \
      CAMLprim value ml_##name(value v1, value v2, value v3) { \
        return retval (name(conv1(v1), conv2(v2), conv3(v3))); }

В caml/alloc.h упоминается полезная функция caml\_convert\_flag\_list.
Ее применяют для удобной работы со списком флагов, которые в С обычно
представляются битами переменной типа int. Ее использование:

    type open_flag =
      | O_RDONLY
      | O_WRONLY
      | O_RDWR;

    external open : string -> open_flag list -> int = "ml_open"

    static int tbl_flag[] = {
      O_RDONLY,
      O_WRONLY,
      O_RDWR
    };
      
    static int myflags(value flags) {
       return caml_conver_flag_list(flags, tbl_flags));
    }

    ML_2(open, String_val, myflags, Val_int)

* * * * *

2011-03-26 13:08
