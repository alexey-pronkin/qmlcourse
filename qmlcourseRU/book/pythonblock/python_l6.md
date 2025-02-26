---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

(python_l6)=

# Списки и циклы в Python

## Описание лекции

На этом занятии мы разберем следующие темы:

- списки (`list`) и их методы;
- индексация списков;
- что такое срезы и зачем они нужны;
- цикл `for` и функция `range`;
- итерация по спискам, list comprehensions.

## Введение в списки объектов
В предыдущих лекциях мы оперировали малым количеством переменных. Для каждого блока логики или примера кода вводилось 3-5 объектов, над которыми осуществлялись некоторые операции. Но что делать, если объектов куда больше? Скажем, вам необходимо хранить информацию об учащихся класса -- пусть это будет рост, оценка по математике или что-либо другое. Не знаю, как вы, но я нахожу крайне неудобным создание 30 отдельных переменных. А если еще и нужно посчитать среднюю оценку в классе!

```python
average_grade = petrov_math + kosareva_math + zinchenko_math + kotenkov_math + ...
average_grade = average_grade / 30
```

Такой код к тому же получается крайне негибким: если количество студентов, как и их состав, изменится, то нужно и формулу переписать, так еще и делитель -- в нашем случае 30 -- изменять.

Часто в программах -- даже в (квантовом) машинном обучении -- приходится работать с большим количеством __однотипных__ переменных. Специально для этого придуманы **массивы** (по-английски array). В Python их еще называют **списками** (list). В некоторых языках программирования эти понятия отличаются, но не в Python. Список может хранить переменные **разного** типа. Также списки называют ["контейнерами"](https://ru.wikipedia.org/wiki/Контейнер_(программирование)), так как они хранят какой-то набор данных. Для создания простого списка необходимо указать квадратные скобки или вызвать конструктор типа (`list` -- это отдельный тип, фактически такой же, как int или str), а затем перечислить **объекты через запятую**:

```{code-cell} ipython3
# разные способы объявления списков
first_list = []
second_list = list()
third_list = list([1,2, "stroka", 3.14])
fourth_lust = [15, 2.2, ["another_list", False]]

print(type(second_list), type(fourth_lust))
print(first_list, fourth_lust)
```

```{tip}
Хоть список и хранит переменные разного типа, но так делать без особой необходимости не рекомендуется -- вы сами скорее запутаетесь и ошибетесь в обработке объектов списка. В большинстве других языков прогрммирования массив может хранить только объекты одного типа.

Для хранения сложных структур (скажем, описание студента -- это не только оценка по математике, но и фамилия, имя, адрес, рост и так далее) лучше использовать классы -- с ними мы познакомимся в будущем. А еще могут пригодиться **кортежи**, или `tuple`. Про них в лекции не рассказано, самостоятельно можно ознакомиться [по ссылке](https://pythonworld.ru/tipy-dannyx-v-python/kortezhi-tuple.html).
```

Теперь можно один раз создать список и работать с ним как с единым целым. Да, по прежнему для заведения оценок студентов придется разово их зафиксировать, но потом куда проще исправлять и добавлять! Рассмотрим пример нахождения средней оценки группы, в которой всего 3 учащихся, но к ним присоединили еще 2, а затем -- целых 5:

```{code-cell} ipython3
# базовый журнал с тремя оценками
math_journal = [3, 3, 5]

# добавим новопришедших студентов
math_journal.append(4)
math_journal.append(5)

# и сразу большую группу новых студентов
math_journal.extend([2,3,4,5,5])

print(f"{math_journal = }")

# найдем среднюю оценку как сумму всех оценок, деленную на их количество
avg_grade = sum(math_journal) / len(math_journal)
print(f"{avg_grade = }")
```

В коде выше продемонстрировано сразу несколько важных аспектов:
1. Добавлять по одному объекту в конец списка можно с помощью метода списка `append`;
2. Метод `append` принимает в качестве аргумента один Python-объект;
3. Слияние списков (конкатенация, прямо как при работе со строками) нескольких осуществляется командой `extend` (расширить в переводе с английского);
4. Для списков определена функция `len`, которая возвращает целое число `int` -- количество объектов в списке;
5. Функция `sum` может применяться к спискам для суммирования всех объектов (если позволяет тип -- то есть для `float`, `int` и `bool`. Попробуйте разобраться самостоятельно, как функция работает с последним указанным типом);
6. Для методов `append` и `extend` не нужно приравнивать результат выполнения какой-то переменной -- изменится сам объект, у которого был вызван метод (в данном случае это `math_journal`);
7. Списки в Python **упорядочены**, то есть объекты сами по себе места не меняют, и помнят, в каком порядке были добавлены в массив.

```{tip}
В тексте выше встречается термин **метод**, который, быть может, вам не знаком. По сути метод -- это такая же **функция**, о которых мы говорили раньше, но она принадлежит какому-то объекту с определенным типом. Не переживайте, если что-то непонятно -- про функции и методы мы поговорим подробно в ближайших лекциях!

`print`, `sum` -- функции, они существуют сами по себе;
`append`, `extend` -- методы объектов класса `list`, не могут использоваться без них.
```

## Индексация списков
Теперь, когда стало понятно, с чем предстоит иметь дело, попробуем усложнить пример. Как узнать, какая оценка у третьего студента? Все просто -- нужно воспользоваться **индексацией** списка:

```{code-cell} ipython3
# базовый журнал с пятью оценками
math_journal = [1, 2, 3, 4, 5]

third_student_grade = math_journal[3]
print(third_student_grade)
```

И снова непонятный пример! Давайте разбираться:
1. Для обращения к `i`-тому объекту нужно в квадратных скобках указать его индекс;
2. **Индекс** в Python начинается **С НУЛЯ** -- это самое важное и неочевидное, здесь чаще всего случаются ошибки;
3. Поэтому `[3]` обозначает взятие **четвертой** оценки (и потому выводится четверка, а не тройка);
4. Всего оценок 5, но так как индексация начинается с нуля, то строчка `math_journal[5]` выведет ошибку -- нам доступны лишь индексы `[0, 1, 2, 3, 4]` для взятия (так называется процедура обращения к элементу списка по индексу -- взятие по индексу).

```{figure} /_static/pythonblock/list_loops_l6/list_indexing_1.png
:name: list_indexing
:width: 400px

Пример списка из трех объектов. Сверху показаны их индексы, включая отрицательные
```

Также в `Python` существуют отрицательные индексы (-1, -2 ...). Они отсчитывают объекты списка, начиная с конца. Так как нуль уже занят (под первый объект), то он не используется.

```{code-cell} ipython3
# базовый журнал с пятью оценками
math_journal = [1, 2, 3, 4, 5]

# возьмем последнюю оценку
last_grade = math_journal[-1]
print(f"Последняя оценка: {last_grade}")

# а теперь -- предпоследнюю
prev = math_journal[-2]
print(f"Предпоследняя оценка: {prev}")

# конечно, взятие по индексам можно использовать в ранее разобранном синтаксисе

if math_journal[-1] < math_journal[-2]:
    math_journal[-1] += 1
    print("Последняя оценка меньше предпоследней. Натянем студенту?")
else:
    math_journal[-2] = 2
    print("Последний студент сдал очень хорошо, на его фоне предпоследний просто двоечник!")
```

Все это важно не только для грамотного оперирования конкретными объектами, но и следующей темы -

## Срезы
Срезы, или slices -- это механизм обращения сразу к нескольким объектам списка. Для создания среза нужно в квадратных скобках указать двоеточие, слева от него -- индекс начала среза (по умолчанию 0, можно не выставлять) **включительно**, справа -- границу среза **не включительно** (пустота означает "до конца списка"). Может показаться нелогичной такая разнородность указания границ, но на самом деле она безумно удобна -- особенно вместе с тем, что индексация начинается с нуля. Быстрее объяснить на примере:

```{code-cell} ipython3
# базовый журнал с пятью оценками
math_journal = [1, 2, 3, 4, 5]

# как взять первые 3 оценки?
first_3_grades = math_journal[:3]
print(f"{first_3_grades = }")

# как взять последние две оценки?
last_2_grades = math_journal[-2:]
print(f"{last_2_grades = }")

# сделаем срез на 4 оценки, начиная со второй (с индексом 1)
start_index = 1
some_slice = math_journal[start_index : start_index + 4]
print(f"{some_slice = }")

# возьмем столько объектов из начала, сколько объектов в some_slice
yet_another_slice = math_journal[:len(some_slice)]

# а вот так можно проверить, попадает ли объект в список
print("Верно ли, что единица входит в some_slice? {1 in some_slice}")
print("Верно ли, что единица входит в yet_another_slice? {1 in yet_another_slice}")
```

```{tip}
Можно сделать пустой срез, и тогда Python вернет пустой список без объектов. Можете проверить сами:
`["1", "2", "3"][10:20]`
```

Давайте проговорим основные моменты, которые **крайне важно понять**:
1. Так как индексация начинается с нуля (значение по умолчанию) и правая граница не включается в срез, то берутся объекты с индексами `[0,1,2]`, что в точности равняется трем первым объектам;
2. Срез `[-2:]` указывает на то, что нужно взять все объекты до конца, начиная с предпоследнего
3. Значения в срезе могут быть **вычислимы** (и задаваться сколь угодно сложной формулой), но должны оставаться **целочисленными**;
4. Если нужно взять `k` объектов, начиная с `i`-го индекса, то достаточно в качестве конца среза указать `k+i`;
5. Для проверки вхождения какого-либо объекта в список нужно использовать конструкцию `x_obj in some_list`, которая вернет `True`, если массив содержит `x_obj`, и `False` в ином случае;
6. Самый простой способ сделать копию списка - это сделать срез по всему объекту: `my_list[:]`. Однако будьте внимательны -- в одних случаях копирование происходит полностью (по значению), а в некоторых сохраняются ссылки (то есть изменив один объект в скопированном списке вы измените объект в исходном). Связано это с типом объектов (mutable/immutable), подробнее об этом будет рассказано в следующей лекции. В общем, если вы работаете с простыми типами (`int`/`str`), то срез вернет копию, и ее изменение не затронет исходный список. Однако для хранения новых данных нужна память, поэтому при копировании десятков миллионов объектов можно получить ошибку, связанную с нехваткой памяти.

## Циклы
До сих пор в примерах мы хоть и обращались к нескольким объектам, добавляли и меняли их, все еще не было рассмотрено взаимодействие сразу с несколькими. Давайте попробуем посчитать, сколько студентов получили оценку от 4 и выше. Для этого интуитивно кажется, что нужно **пройтись по всем оценкам от первой до последней**, сравнить каждую с четверкой. Для прохода по списку, или **итерации**, используются **циклы**.
Общий синтаксис таков:

```python
example_list = list(...)
for item in example_list:
    <> блок кода внутри цикла (аналогично блоку в if)
    ... что-то сделать с item
    <>
```

Здесь `example_list` -- это некоторый итерируемый объект. Помимо списка в Python существуют и другие итерируемые объекты, но пока будем говорить о массивах.

Этот цикл работает так: указанной **переменной `item` присваивается первое значение из списка**, и выполняется **блок кода** внутри цикла (этот блок, напомним, определяется отступом. Он выполняется весь от начала отступа и до конца, как и было объяснено в пятой лекции). Этот блок еще иногда называют **телом цикла**. Потом переменной `item` присваивается следующее значение (второе), и так далее. Переменную, кстати, можно называть как угодно, не обязательно `item`.

**Итерацией** называется каждый **отдельный проход** по телу цикла. Цикл всегда повторяет команды из тела цикла несколько раз. Два примера кода ниже аналогичны:

```{code-cell} ipython3
math_journal = [3, 4, 5]
counter = 0

for cur_grade in math_journal:
    if cur_grade >= 4:
        counter += 1

print(f"Всего хорошистов и отличников по математике {counter} человека")
```

```{code-cell} ipython3
math_journal = [3, 4, 5]
counter = 0

cur_grade = math_journal[0]
if cur_grade >= 4:
    counter += 1

# не забываем менять индекс с 0 на 1, так как каждый раз берется следующий элемент
cur_grade = math_journal[1]
if cur_grade >= 4:
    counter += 1

# и с единицы на двойку
cur_grade = math_journal[2]
if cur_grade >= 4:
    counter += 1

print(f"Всего хорошистов и отличников по математике {counter} человека")
```

Понятно, что первый кусок кода обобщается на любой случай -- хоть оценок десять, хоть тысяча. Второе решение не масштабируется, появляется **много одинакового кода, в котором легко ошибиться** (не поменять индекс, к примеру).

Движемся дальше. Так как каждый элемент списка закреплен за конкретным индексом, то в практике часто возникают задачи, логика которых завязана на индексах. Это привело к тому, что появилась альтернатива для итерации по списку. Функция `range` принимает аргументы, аналогичные срезу в списке, и возвращает итерируемый объект, в котором содержатся целые числа (индексы). Так как аргументы являются аргументами функции, а не среза, то они соединяются запятой (как `print(a, b)` нескольких объектов). Если подан всего один аргумент, то нижняя граница приравнивается к нулю. Посмотрим на практике, как сохранить номера (индексы) всех хорошо учащихся студентов:

```{code-cell} ipython3
math_journal = [4, 3, 4, 5, 5, 2, 3, 4]
good_student_indexes = []

for student_index in range(len(math_journal)):
    curent_student_grade = math_journal[student_index]
    if curent_student_grade >= 4:
        good_student_indexes.append(student_index)

print(f"Преуспевающие по математике студенты находятся на позициях: {good_student_indexes}")
```

В примере `student_index` принимает последовательно все значения от `0` до `7` включительно. `len(math_journal)` равняется `8`, а значит, восьмерка сама не будет включена в набор индексов для перебора. На каждой итерации `curent_student_grade` меняет свое значение, после чего происходит проверка. Если бы была необходимость пробежаться только по студентам, начиная с третьего, то нужно было бы указать `range(2, len(math_journal))` (двойка вместо тройки потому, что индексация с нуля, ведь мы перебираем индексы массива).

Выше описаны основные концепции обращения со списками. Их крайне важно понять и хорошо усвоить, без этого писать любой код будет безумно сложно. Скопируйте примеры к себе в `.ipynb`-ноутбук, поиграйтесь, поменяйте параметры цикла и проанализируйте изменения.

## List comprehensions
Некоторые циклы настолько просты, что занимают 2 или 3 строчки. Как пример -- привести список чисел к списку строк:

```{code-cell} ipython3
# грубый вариант
inp_list = [1,4,6,8]
out_list = []

for item in inp_list:
    out_list.append(str(item))

# list comprehension
out_list = [str(item) for item in inp_list]
print(out_list)
```

Две части кода идентичны за вычетом того, что нижняя -- с непонятной конструкцией в скобках -- короче. Python позволяет в рамках одной строки произвести какие-либо простые преобразования (помним, что `str()` -- это вызов функции, а значит если у вас есть сложная функция, которая делает квантовые вычисления, то ее также можно применить!). Фактически самый частый пример использования -- это паттерн "**применение функции к каждому объекту списка**".

## Что мы узнали из лекции
- `list` -- это **объект-контейнер, который хранит другие объекты разных типов**. Запись происходит упорядочено и последовательно, а каждому объекту присвоен **целочисленный номер, начиная с нуля**;
- для добавления одного объекта в `list` нужно использовать метод объекта `list` -- `append`, а для расширения списка сразу на несколько позиций пригодится `extend`;
- проверить, сходит ли объект в список, можно с помощью конструкции `obj in some_list`;
- индексы **могут быть отрицательными**: `-1`, `-2` ... В таком случае нумерация начинается от последнего объекта;
- можно получить часть списка, сделав **срез** с помощью конструкции `list[start_index : end_index]`, при этом объект на позиции `end_idnex` не будет включен в возвращаемый список (т.е. **срез работает не включительно по правую границу**);
- часто со списками используют **циклы, которые позволяют итерироваться по объектам массива** и выполнять произвольную логику в рамках отделенного отступом блока кода;
- для итерации по индексам можно использовать `range()`;
- простые циклы можно свернуть в **list comprehension**, и самый частый паттерн для такого преобразования -- это **применение некоторой функции к каждому объекту** списка (если `x` это функция, то синтаксис будет таков: `[x(item) for item in list])`).
