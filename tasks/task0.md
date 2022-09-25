Задача 0. Slice-n-dice
========================

## Подготовка

Вспомните, что такое шаблоны, явные специализации (полные и частичные).

Вспомните, что шаблон `std::span` стандартной библиотеки используется как невладеющая ссылка на несколько подряд расположенных в памяти объектов. Он служит "наибольшим общим знаменателем" между различными видами [contiguous контейнеров](https://en.cppreference.com/w/cpp/named_req/ContiguousContainer): используя его в качестве аргумента функции вы позволяете пользователю передавать в вашу функцию и `std::vector` от произвольного аллокатора, и `std::array` любого размера, и различные обёртки вроде [`std::flat_map`](https://en.cppreference.com/w/cpp/header/flat_map), и даже контейнеры из других библиотек, коих на практике встречается великое множество.

Вспомните про существование операторов каста.

Обратите внимание, что у `std::span` не один шаблонный аргумент, а два: тип элемента и размер. Размер может быть задан как std::dynamic_extent, что означает "размер не известен во время компиляции, но будет известен в рантайме".

## Задача

### Slice

В данной задаче вам предстоит написать более общую версию `std::span`: `Slice`, поддерживающий дополнительный шаблонный параметр stride -- размер шага при переходе к следующему элементу. Например,
```c++
    std::array<int, 10> data;
    std::iota(data.begin(), data.end(), 0);
    slice<int, 5, 2> even_elements{data};
```
Здесь содержимое `even_elements` должно быть `0, 2, 4, 6, 8`. Также ваша имплементация должна поддерживать рантайм значение как обоих параметров, так и любого одного из них, при подстановке в шаблон `std::dynamic_extents`.

### Интерфейс Slice

Требования к шаблону `Slice` -- повторение интерфейса [`std::span`](https://en.cppreference.com/w/cpp/container/span), кроме следующих исключений:
* `size_bytes`
* `as_bytes`
* `as_writable_bytes`
* `subspan`
Эти методы подразумевают `stride == 1`, поэтому пока что мы будем их игнорировать. Также пока что игнорируйте раздел "Helper templates". Наконец, конструкторы `std::span` -- достаточно тонкая наука, поэтому предлагается не повторять их один к одному с cppreference, а написать свои с нуля так, чтобы тесты проходили.
Далее, нам потребуется некоторое количество дополнений связанных с параметром `stride`:
* Методы `Skip`, позволяющие поменять `stride`
* Методы `DropFirst`, откидывающие несколько первых элементов
* Методы `DropLast`, откидывающие несколько последних элементов
* Модификация конструктора от итератора и размера, принимающая ещё и `stride`
Обратите внимание, что мы не требуем метода `Subslice`, аналогичного `subspan`. Попытка написать такой метод, учитывающий все комбинации рантайм и компайлтайм параметров, ведёт к комбинаторному взрыву, поэтому вместо него мы предоставляем более элементарные методы `First`, `Last`, `DropFirst`, `DropLast` и `Stride`.

Ещё мы требуем, чтобы `Slice` со статическими параметрами неявно кастился к аналогичному слайсу с динамическими параметрами (и оба одновременно, и только один по-отдельности). Каст `Slice<T>` к `Slice<const T>` также должен быть неявным.

Также обязательное требование -- размер каждой специализации `Slice` должен быть минимальным из возможных. То есть нельзя хранить данные об `extent` и `stride` в рантайме, если они известны в компайлтайме. Подумайте как добиться этого не решив всю задачу заново 4 раза. Возможно вам поможет наследование? Не бойтесь использовать всю силу C++ и хоть полностью менять шаблон решения. Главное -- пройти тесты.

Не засоряйте глобальный неймспейс вспомогательными шаблонами, помещайте их в отдельный `namespace`.

Также не забудьте ознакомиться с [тестами](https://github.com/Mrkol/metaprogramming-course/blob/master/tests/task0).

## Формальности

**Дедлайн:** 04:00 3.10.2022.

**Баллы:** 2 уе.

Шаблон `Slice` должнен быть доступен в глобальном неймспейсе при подключении заголовочного файла `slice.hpp`. Обратите внимание, что создание дополнительных файлов и классов не возбраняется. `cpp` файлы в папке будут автоматически скомпилированы и прилинкованы к тестам, хоть в этой задаче они скорее всего и не пригодятся.

Код пушьте в ветку `task0` и делайте pull request в `master`.