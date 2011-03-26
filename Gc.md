![image](logo.png)
-   Gc

\
\
 [[LanguageSetup](LanguageSetup.html)] [[TitleIndex](TitleIndex.html)]
[[WordIndex](WordIndex.html)]

* * * * *

## Устройство

Generational. Два поколения. Minor и major heap. Ref list. Три типа
сборки - minor + major slice, major, compaction. Gc не лазит за пределы
heap'ов, использует тэги в заголовках блоков для определения структуры
данных для обхода.

## Тюнинг

Имеет смысл если профайлинг показывает что программа большую часть
времени проводит в сборщике мусора. В качестве меры работы Gc можно
использовать отношение *promoted words* к *minor words* (чем меньше, тем
лучше).

1.  По-умолчанию (в ocaml <= 3.12.0) размер *minor heap* слишком
    маленький (для современных систем). Рекомендуется
    поэкспериментировать в сторону увеличения.
2.  Аналогично *major heap increment* (на сколько увеличивается *major
    heap*).
3.  Рантайм вызывает major gc основываясь на оценках кол-ва мусора в
    heap'е. Эта оценка регулируется параметром *space\_overhead*.
4.  Compaction вызывается основываясь на *max\_overhead*.

Cм. [документацию на модуль
Gc](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Gc.html).

Огромные долгоживущие структурно-неизменяемые массивы данных при каждом
цикле gc будут проверяться мусорщиком. Варианты решения :

-   уменьшить кол-во циклов gc (как описано выше)
-   убрать данные из ocaml heap (и освобождать память вручную, см.
    [Ancient](http://merjis.com/developers/ancient))
-   массивам явно не содержащим указателей (например *int array*)
    вручную поставить No\_scan\_tag (теоретический непроверенный на
    практике хак).

## See also

-   [Beginner's guide to OCaml
    internals](http://rwmj.wordpress.com/2009/08/04/ocaml-internals/)
-   [Christoph Spiel: [CIL users] CIL perfomance
    issues](http://www.mail-archive.com/cil-users@lists.sourceforge.net/msg00070.html)

* * * * *

2011-03-26 13:09
