WTF
===

Мало библиотек
--------------

Чего не хватает?


Нет готовых сборок под windows
------------------------------

* новая официальная сборка, ссылка
* сборка от ocamlpro
* build.ygrek.org.ua
* инструкция по сборке на mingw - проверить
* https://github.com/protz/ocaml-installer
* что с x64?


Нет инфраструктуры для сборки пакетов вместе с зависимостями
------------------------------------------------------------

Это правда. Но существуют частичные решения проблемы:

### [OASIS-DB](http://oasis.ocamlcore.org/)

Неспеша развивающийся проект по созданию CPAN-подобного каталога проектов, для
OCaml, особенности (*pitfalls*? :)

  * исходный код проектов хранятся на сервере OASIS-DB
  * проекты добавляются ментейнерами вручную, поэтому содержимое трех имеющихся
    репозиториев (stable, testing, unstable) практически не изменяется.
  * предполагается что в будущем любой желающий сможет написать свой велосипед,
    для установки пакетов, используя API OASIS-DB
  * на данный момент единственный скрипт умеющий это --
    [odb.ml](https://github.com/thelema/odb)

Пример использования ``odb.ml``:

    $ odb --repo unstable
    Available packages: ANSITerminal CameraRescue CamlGI MOIFile archimedes archive batteries bench benchmark bin_prot cairo2 calendar camomile cmdliner cryptokit csv csv-analyze csv-generate csvgenerator curl estring expect extlib extunix fastrandom fieldslib fileutils gsl hdf4 irrlicht janest-core janest-core_extended lambda-term lambda-term-actions lbfgs lpd lwt monad-custom mysql oUnit oasis ocamlgraph ocamlify ocamlmod ocamlscript ocsigen-bundler odn pcre planck posix_resource radixtree react sexplib spotlib sqlexpr sqlite3 text type-conv utop utop-emacs xdg-basedir xmlm xstrp4 zarith zed zip
    $ odb lwt zip zed
    ... installing packages ...


### GODI

### rebar of OCaml

Учитывая всю печальность ситуации, ожидается что какой нибудь доблестный
велосипедист из рядов camlunity возьмет и запилит человеческую альтернативу
всему вышеперечисленному, основные моменты :

* [обсуждение в чате](http://chatlogs.jabber.ru/ocaml@conference.jabber.ru/2011/11/15.html#12:46:25.86714)
* [план](https://docs.google.com/document/d/1dxbuu3RP3NCxMI54YFRtnwFUOTrIj-EWmBdwy32a0CI/edit) в google docs

Эта вики уже достаточно полезна, но малоизвестна и недописана
-------------------------------------------------------------

Нет англоязычной вики
---------------------

Недостатки реализации
=====================

Ограниченная многопоточность
----------------------------

   альтернативный рантайм - oc4mc

   альтернативная архитектура приложения - message passing :

   * [jocaml](http://jocaml.inria.fr/)

   * [parvel](https://bitbucket.org/gds/parvel/)

   * etc

   NB асинхронность <> паралельность, для асинхронности есть

   * lwt

   * janest Async

   * etc

Фундаментальные ограничения языка
=================================

Отсутствует generic print
-------------------------

  есть [патч](http://www.dimino.org/ocaml-3.12.1-generic-print.patch)

Не самый лучший оптимизатор
---------------------------
