* * * * *

Подробный оригинальный changelog из svn : http://caml.inria.fr/svn/ocaml/trunk/Changes

Здесь же краткий обзор основных изменений.

# 4.00.0 (2012-07-26)

- GADT'ы (generalized abstract datatypes)
	([официальный	мануал](http://caml.inria.fr/pub/distrib/ocaml-4.00/ocaml-4.00beta-refman.html#htoc113),
	[страница разработчиков](https://sites.google.com/site/ocamlgadt/)).
- Меньше аннотаций типов при создании и распаковке first-class модулей (компилятор выводит типы
	из контекста).
- Новый синтаксис `(module M)` и `(module M : S)` в паттернах сопоставления, для распаковки
	first-class модулей на месте.
- Начисто переписан бэкенд для ARM процессоров: поддержка архитектур armel (EABI) и armhf
	(EABI+VFPv3), Thumb-2 инструкции, PIC, natdynlink, профайлинг и бэктрейсы.
- Установка внутренних библиотек компилятора в `+compiler-libs`.
- Новый инсталлятор на Windows ([страница разработки](http://protz.github.com/ocaml-installer/)).
- Больше оптимизаций:
	- revised simplification of let-alias ([PR#5205](http://caml.inria.fr/mantis/view.php?id=5205),
		[PR#5288](http://caml.inria.fr/mantis/view.php?id=5288))
	- избавление от бета-редукций на этапе компиляции e.g. `(fun x y -> e) a b`
	- меньше аллокаций при вызове частично-применённых функций ([PR#5287](http://caml.inria.fr/mantis/view.php?id=5287))
	- все оптимизации выполняются при использовании `ocamlopt -g` ([PR#5313](http://caml.inria.fr/mantis/view.php?id=5313))
- Больше отладочной информации для x86 и amd64: cfi аннотации (точные нативные бэктрейсы), имена
	файлов и номера строк (точки останова) ([PR#5487](http://caml.inria.fr/mantis/view.php?id=5487)).
- Новые предпреждения компилятора о бесполезных `open` и неиспользуемых именах
	([PR#5437](http://caml.inria.fr/mantis/view.php?id=5437),
	[PR#5438](http://caml.inria.fr/mantis/view.php?id=5438)).
- Новая реализация модуля `Hashtbl` : статистически лучшая хэш-функция, опциональная рандомизация 
	([PR#5222](http://caml.inria.fr/mantis/view.php?id=5222),[PR#5225](http://caml.inria.fr/mantis/view.php?id=5225)).
- Маршаллинг функций из динамически-подгружаемого кода
	([PR#5215](http://caml.inria.fr/mantis/view.php?id=5215)).
- `Random.self_init ()` теперь использует `/dev/urandom`, если возможно
- Более надёжное определение переполнения стека при вызове сишного кода (Linux)
	([PR#4746](http://caml.inria.fr/mantis/view.php?id=4746),
	[PR#5064](http://caml.inria.fr/mantis/view.php?id=5064),
	[PR#5485](http://caml.inria.fr/mantis/view.php?id=5485)).
- Новые примитивы `%apply` и `%revapply` ([PR#5236](http://caml.inria.fr/mantis/view.php?id=5236)).
- Удалены неподдерживаемые нативные кодогенераторы для архитектур Alpha, HPPA, IA64, MIPS.
- Удалены из официального дистрибутива библиотека DBM и топлевел OcamlWin.

