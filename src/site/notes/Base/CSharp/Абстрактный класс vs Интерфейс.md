---
{"dg-publish":true,"permalink":"/base/c-sharp/abstraktnyj-klass-vs-interfejs/"}
---

Когда использовать абстрактный класс:
1. Общая функциональность для родственных объектов
2. Когда в перспективе может быть большое количество базового функционала для дочерних объектов

Интерфейс определяет некоторый функционал который должен реализовать его наследник 

Когда использовать интерфейсы:
1. Для определения некого функционала для группы разрозненных объектов, при этом оставляя детали реализации за ними

**Интерфейсы помогают нам добиться взаимозаменяемости** частей класса

К примеру я работал с библиотекой для просмотра видео c ip камер. Приводя общий класс к определённому интерфейсу я точно знал что делать, интерфейс служил только определённому функционалу, там были методы которые служили определенному действию, не составляла труда разобраться с логикой этих паре методов. Но если я на этапе разработки не использовались интерфейсы, мне пришлось бы работать с общим классом, который буквально содержал сотни методов, разобраться в них было бы очень проблематично