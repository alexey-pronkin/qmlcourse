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

(classicsvm)=

# Классический SVM

## Описание лекции

В этой лекции мы рассмотрим классический метод опорных векторов (SVM) и разберем стоящую за ним математику. С согласия Евгения Соколова, мы во многом переиспользуем конспекты лекций [курса](https://github.com/esokolov/ml-course-hse) "Машинное обучение", читаемого на ФКН ВШЭ.

План лекции такой:

- линейная классификация;
- интуиция метода опорных векторов;
- метод опорных векторов для линейно-разделимой выборки;
- метод опорных векторов для линейно-неразделимой выборки;
- решение задачи метода опорных векторов;
- ядерный переход;
- плюсы и минусы SVM.

## Линейная классификация

Основная идея линейного классификатора заключается в том, что признаковое пространство может быть разделено гиперплоскостью на два полупространства, в каждом из которых прогнозируется одно из двух значений целевого класса. Если это можно сделать без ошибок, то обучающая выборка называется _линейно разделимой_.

```{figure} /_static/qsvmblock/lin_classifier.png
:width: 350px
:name: lin_classifier

Линейная классификация
```

Рассмотрим задачу бинарной классификации, в которой $\mathbb{X} = \mathbb{R}^d$ -- пространство объектов,
$Y = \{-1, +1\}$ -- множество допустимых ответов (целевой признак),
$X = \{(x_i, y_i)\}_{i = 1}^\ell$ -- обучающая выборка. Будем класс $+1$ называть положительным, а класс $-1$ -- отрицательным. Здесь $d$ -- размерность признакового пространства, $\ell$ -- количество примеров в обучающей выборке.

__Линейная модель классификации__ определяется следующим образом:

$$
    a(x) =
    \text{sign} \left(
        \langle w, x \rangle + b
    \right)
    =
    \text{sign} \left(
        \sum_{j = 1}^{d} w_j x_j + b
    \right),
$$

где $w \in \mathbb{R}^d$ -- вектор весов, $b \in \mathbb{R}$ -- сдвиг (bias), $\text{sign}(\cdot)$ – функция "сигнум", возвращающая знак своего аргумента, $a(x)$ – ответ классификатора на примере $x$.

```{note}

Хорошо бы для векторов ставить стрелку и писать $\vec{w}$, но мы этого не будем делать, предполагая, что из контекста ясно, что вектор, а что скаляр. В частности, в формуле выше $w$ и $x$ -- вектора, а $a(x), w_j, x_j, d$ и $b$ -- скаляры.
```

Часто считают, что среди признаков
есть константа, $x_{d + 1} = 1$.
В этом случае нет необходимости вводить сдвиг $b$,
и линейный классификатор можно задавать как

$$
    a(x) = \text{sign} \langle w, x \rangle.
$$


```{figure} /_static/qsvmblock/distance_to_the_plane.png
:width: 400px
:name: distance_to_the_plane

Расстояние от точки до плоскости
```

Геометрически линейный классификатор соответствует гиперплоскости с вектором нормали $w$, которая задается уравнением $\langle w, x \rangle = 0$. Величина скалярного произведения $\langle w, x \rangle$ пропорциональна расстоянию от гиперплоскости до точки $x$, а его знак показывает, с какой стороны от гиперплоскости находится данная точка. Если быть точным, расстояние от точки с радиус-вектором $x_A$ до плоскости $\langle w, x \rangle = 0$:

$$
\rho(x_A, \langle w, x \rangle = 0) = \frac{\langle w, x_A \rangle}{||w||}.
$$

```{admonition} Упражнение на метод Лагранжа

[Метод Лагранжа](https://en.wikipedia.org/wiki/Lagrange_multiplier) -- очень важный метод оптимизации функций при наличии органичений. Этот же метод вовсю используется ниже в методе опорных векторов.

Рекомендуется ознакомиться с методом Лагранжа и решить (ручками!) простую оптимизационную задачу вида

$$
\begin{equation}
    \left\{
        \begin{aligned}
            & x + y \to \min_{x,y} \\
            & x^2 + y^2 = 1
        \end{aligned}
    \right.
\end{equation}
$$

А мы выведем указанную выше формулу расстояния от точки до плоскости. Вообще это можно сделать разными способами -- алгебраически и геометрически. Но давайте посмотрим на эту задачу (внезапно) как на задачу оптимизации и решим ее методом Лагранжа. Это послужит неплохой тренировкой перед тем как окунуться в SVM.

Итак, представим задачу в таком виде: хотим найти точку $x$ на плоскости $\langle w, x \rangle = 0$ такую, что расстояние от $x$ до точки $x_0$ минимально:

$$
\begin{equation}
    \left\{
        \begin{aligned}
            & \rho(x, x_0) \to \min_{x} \\
            & \langle w, x \rangle = 0
        \end{aligned}
    \right.
\end{equation}
$$


Лагранжиан: $\mathcal{L}(x, \lambda) = {||x - x_0||}^2 + 2 \lambda \langle w, x \rangle$. Тут мы для $\rho(x, x_0)$ подставили квадрат расстояния ${||x - x_0||}^2$ (такой переход от росстояния к его квадрат хорошо бы обосновать монотонностью оптимизируемой функции, но мы это опустим), а также перед коэффициентом $\lambda$ для удобства поставили 2, что несущественно.

Далее надо приравнять нулю частные производные лагранжиана по его аргументам. Из
$\frac{\partial \mathcal{L}}{\partial x} = 0$ получаем: $2(x - x_0) + 2 \lambda w = 0 \Rightarrow x = x_0 - \lambda w$.

Теперь домножим это уравнение скалярно на $w$ и выразим $\lambda$:

$$
\langle w, x \rangle  = \langle w, x_0 \rangle - \lambda {||w||}^2 \Rightarrow \lambda = \frac{\langle w, x_0 \rangle}{{||w||}^2 }
$$

Тогда наконец

$$
\min_{x,  \langle w, x \rangle = 0} \rho(x, x_0) = ||x - x_0|| = ||(x_0 - \lambda w) - x_0|| = |\lambda| ||w|| = \frac{\langle w, x_0 \rangle}{||w||}\ \ \square.
$$

```

Таким образом, линейный классификатор разделяет пространство на две части с помощью гиперплоскости, и при этом одно полупространство относит к положительному классу, а другое -- к отрицательному.

Пожалуй, самый известный и популярный на практике представитель семейства линейных классификаторов -- логистическая регрессия. На русском языке про нее можно почитать в [статье](https://habr.com/ru/company/ods/blog/323890/) открытого курса по машинному обучению или в упомянутых [лекциях](https://github.com/esokolov/ml-course-hse/tree/master/2021-fall/lecture-notes) Евгения Соколова. В этих лекциях также объясняется, как происходит обучение модели (подбор весов $w$) за счет минимизации функции потерь.

### Отступ классификации

Оказывается полезным рассмотреть выражение $M(x_i, y_i, w) = y_i \cdot \langle w, x \rangle.$

Это __отступ классификации__ (margin) для объекта обучающей выборки $(x_i, y_i)$. К сожалению, его очень легко перепутать с зазором классификации, который появится чуть ниже в изложении интуиции метода опорных векторов. Чтобы не путать: отступ определен на конкретном объекте обучающей выборки.


```{figure} /_static/qsvmblock/margin_toy_example.png
:width: 400px
:name: margin_toy_example

Иллюстрация к понятию отступа классификации
```

Отступ -- это своего рода "уверенность" модели в классификации объекта $(x_i, y_i)$:

- если отступ большой (по модулю) и положительный, это значит, что метка класса поставлена правильно, а объект находится далеко от разделяющей гиперплоскости (такой объект классифицируется уверенно). На рисунке – $x_3$.
- если отступ большой (по модулю) и отрицательный, значит метка класса поставлена неправильно, а объект находится далеко от разделяющей гиперплоскости (скорее всего такой объект – аномалия, например, его метка в обучающей выборке поставлена неправильно). На рисунке – $x_1$.
- если отступ малый (по модулю), то объект находится близко к разделяющей гиперплоскости, а знак отступа определяет, правильно ли объект классифицирован. На рисунке – $x_2$ и $x_4$.

Далее увидим, что понятие отступа классификации -- часть функции потерь, которая оптимизируется в методе опорных векторов.

## Интуиция метода опорных векторов

Метод опорных векторов (Support Vector Machine, SVM) основан на идее максимизации зазора между классами. Пока не вводим этот термин формально, но передадим интуицию метода. На Рис. {numref}`linclass_margins` показана линейно-разделимая выборка, кружки соответствуют положительным примерам, а квадраты -- отрицательным (или наоборот), а оси -- некоторым признакам этих примеров. На рисунке слева показаны две прямые (в общем случае -- гиперплоскости), разделяющие выборку. Кажется, что синяя прямая лучше тем, что она дальше отстоит от примеров обучающей выборки, чем красная прямая (зазор -- больше), и потому лучше будет разделять другие примеры из того же распределения, что и примеры обучающей выборки. То есть такой линейный классификатор будет лучше обобщаться на новые данные. Теория подтверждает описанную интуицию {cite}`MohriRostamizadehTalwalkar18`.


```{figure} /_static/qsvmblock/linclass_margins.png
:width: 450px
:name: linclass_margins

Слева показаны две прямые (в общем случае -- гиперплоскости), разделяющие выборку. Справа показана прямая, максимизирующая зазор между классами. Источник: [лекция](https://www.cs.cornell.edu/courses/cs4780/2018fa/lectures/lecturenote09.html) Cornell
```

```{note}
Одним из ключевых авторов алгоритма SVM является Владимир Вапник -- советский и американский (с 1991-го года) ученый, который также сделал огромный вклад в теорию классического машинного обучения. Его имя носит одно из ключевых теоретических понятий машинного обучения -- размерность Вапника-Червоненкиса.
```

## Метод опорных векторов для линейно-разделимой выборки

Будем рассматривать линейные классификаторы вида

$$ a(x) = \text{sign} (\langle w, x \rangle + b), \qquad w \in \mathbb{R}^d, b \in \mathbb{R}. $$

Заметьте, что мы вернули сдвиг (bias) $b$. Будем считать, что существуют такие параметры $w_*$ и $b_*$, что соответствующий им классификатор $a(x)$ не допускает ни одной ошибки на обучающей выборке. В этом случае говорят, что выборка __линейно разделима__.


Заметим, что если одновременно умножить параметры $w$ и $b$ на одну и ту же положительную константу, то классификатор не изменится. Распорядимся этой свободой выбора и отнормируем параметры так, что

$$
    \min_{x \in X} | \langle w, x \rangle + b| = 1.
$$

Как мы увидели выше, расстояние от произвольной точки $x_0 \in \mathbb{R}^d$ до гиперплоскости, определяемой данным классификатором, равно

$$
    \rho(x_0, a)
    =
    \frac{
        |\langle w, x \rangle + b|
    }{
        \|w\|
    }.
$$

Тогда расстояние от гиперплоскости до ближайшего примера из обучающей выборки равно

$$
    \min_{x \in X}
    \frac{
        |\langle w, x \rangle + b|
    }{
        \|w\|
    }
    =
    \frac{1}{\|w\|} \min_{x \in X} |\langle w, x \rangle + b|
    =
    \frac{1}{\|w\|}.
$$

Данная величина также называется __зазором__ (margin). Опять же, эту величину очень легко перепутать с отступом классификации, про который мы говорили выше и который тоже margin в англоязычном варианте. Заметим, что отступ -- функция параметров $w$ и конкретного примера обучающей выборки $(x_i, y_i)$, а зазор – функция только параметров $w$ при описанном трюке с нормировкой $w$ и $b$ (в противном случае зазор -- также функция сдвига $b$ и всех примеров $x$).

Таким образом, если классификатор без ошибок разделяет обучающую выборку, то ширина его разделяющей полосы равна $\frac{2}{\|w\|}$.
Максимизация ширины разделяющей полосы приводит
к повышению обобщающей способности классификатора {cite}`MohriRostamizadehTalwalkar18 `. На повышение обобщающей способности направлена и регуляризация,
которая штрафует большую норму весов -- а чем больше норма весов,
тем меньше ширина разделяющей полосы.

Итак, требуется построить классификатор, идеально разделяющий обучающую выборку, и при этом имеющий максимальный отступ.

Запишем соответствующую оптимизационную задачу,
которая и будет определять метод опорных векторов для линейно разделимой выборки (hard margin support vector machine):

$$
\begin{equation}
\label{eq:svmSep}
    \left\{
        \begin{aligned}
            & \frac{1}{2} \|w\|^2 \to \min_{w, b} \\
            & y_i \left(
                \langle w, x_i \rangle + b
            \right) \geq 1, \quad i = 1, \dots, \ell.
        \end{aligned}
    \right.
\end{equation}
$$

Здесь мы воспользовались тем, что линейный классификатор дает правильный ответ на примере $x_i$ тогда и только тогда, когда $y_i (\langle w, x_i \rangle + b) \geq 0$ (вспомним, что $M_i = y_i (\langle w, x_i \rangle + b)$ -- отступ классификации для примера $(x_i, y_i)$ обучающей выборки). Более того, из условия нормировки $\min_{x \in X} | \langle w, x \rangle + b| = 1$ следует, что $y_i (\langle w, x_i \rangle + b) \geq 1$.

В данной задаче функционал является строго выпуклым, а ограничения линейными, поэтому сама задача является выпуклой и имеет единственное решение. Более того, задача является квадратичной и может быть решена крайне эффективно.

## Метод опорных векторов для линейно-неразделимой выборки

Рассмотрим теперь общий случай, когда выборку
невозможно идеально разделить гиперплоскостью.
Это означает, что какие бы $w$ и $b$ мы ни взяли,
хотя бы одно из ограничений в предыдущей задаче
будет нарушено:

$$
    \exists \ x_i \in X:\
    y_i \left(
        \langle w, x_i \rangle + b
    \right) < 1.
$$

Сделаем эти ограничения _мягкими_, введя штраф $\xi_i \geq 0$ за их нарушение:

$$
    y_i \left(
        \langle w, x_i \rangle + b
    \right) \geq 1 - \xi_i, \quad i = 1, \dots, \ell.
$$

Отметим, что если отступ объекта лежит между нулем и
единицей ($0 \leq y_i \left( \langle w, x_i \rangle + b \right) < 1$), то объект верно классифицируется, но имеет ненулевой штраф $\xi > 0$. Таким образом, мы штрафуем объекты за попадание внутрь разделяющей полосы.

Величина $\frac{1}{\|w\|}$ в данном случае называется __мягким зазором__ (soft margin). С одной стороны, мы хотим максимизировать зазор, с другой -- минимизировать
штраф за неидеальное разделение выборки $\sum_{i = 1}^{\ell} \xi_i$.

Эти две задачи противоречат друг другу: как правило, излишняя подгонка под выборку приводит к маленькому зазору, и наоборот -- максимизация зазора приводит к большой ошибке на обучении.
В качестве компромисса будем минимизировать взвешенную сумму двух указанных величин.

Приходим к оптимизационной задаче, соответствующей методу опорных векторов для линейно неразделимой выборки (soft margin support vector machine):

$$
\begin{equation}
\label{eq:svmUnsep}
    \left\{
        \begin{aligned}
            & \frac{1}{2} \|w\|^2 + C \sum_{i = 1}^{\ell} \xi_i \to \min_{w, b, \xi} \\
            & y_i \left(
                \langle w, x_i \rangle + b
            \right) \geq 1 - \xi_i, \quad i = 1, \dots, \ell, \\
            & \xi_i \geq 0, \quad i = 1, \dots, \ell.
        \end{aligned}
    \right.
\end{equation}
$$

Чем больше здесь параметр $C$, тем сильнее мы будем настраиваться на обучающую выборку. Данная задача также является выпуклой и имеет единственное решение.

### Сведение к безусловной задаче оптимизации

Покажем, что задачу метода опорных векторов можно свести к задаче безусловной оптимизации функционала, который имеет вид верхней оценки на долю неправильных ответов.

Перепишем условия задачи:

$$
    \left\{
    \begin{aligned}
        &\xi_i \geq 1 - y_i (\langle w, x_i \rangle + b) \\
        &\xi_i \geq 0
    \end{aligned}
    \right.
$$

Поскольку при этом в функционале требуется, чтобы штрафы $\xi_i$ были как можно меньше, то можно получить следующую явную формулу для них:

$$
    \xi_i
    =
    \max(0,
        1 - y_i (\langle w, x_i \rangle + b)).
$$

Данное выражение для $\xi_i$ уже учитывает в себе все ограничения задачи, описанной выше.
Значит, если подставить его в функционал, получим безусловную задачу оптимизации:

$$
    \frac{1}{2} \|w\|^2
    +
    C
    \sum_{i = 1}^{\ell}
        \max(0,
            1 - y_i (\langle w, x_i \rangle + b))
    \to
    \min_{w, b}
$$

Эта задача является негладкой, поэтому решать её может быть достаточно тяжело. Тем не менее, она показывает, что метод опорных векторов, по сути, как и логистическая регрессия, строит верхнюю оценку на долю ошибок и добавляет к ней стандартную квадратичную регуляризацию. Только если в случае логистической регрессии этой верхней оценкой была логистическая функция потерь (опять сделаем отсылку к [статье](https://habr.com/ru/company/ods/blog/323890/) из открытого курса машинного обучения), то в случае метода опорных векторов это функция вида  $L(y, z) = \max(0, 1 - yz)$, которая называется __кусочно-линейной функцией потерь (hinge loss)__.

```{figure} /_static/qsvmblock/loss_functions_logistic_and_hinge.png
:width: 450px
:name: loss_functions_logistic_and_hinge

Пороговая, кусочно-линейная и логистическая функции потерь
```

Это становится понятнее в терминах упомянутого выше отступа $M_i = y_i (\langle w, x_i \rangle + b)$ на примере обучающей выборки. В идеале мы хотели бы штрафовать классификатор за ошибку на примере: $L_{1/0} (M_i) = [M_i < 0]$. Это пороговая функция потерь (zero-one loss), ее график изображен черным на {numref}`loss_functions_logistic_and_hinge` как функция от отступа. К сожалению, напрямую мы не можем эффективно оптимизировать такую функцию градиентными методами из-за разрыва в нуле, поэтому оптимизируется верхняя оценка zero-one loss. В случае логистической регрессии -- логистическая функция потерь $L(M_i) = \log(1+ e^{-M_i})$ (красная на рисунке выше), а в случае метода опорных векторов -- кусочно-линейная функция $L(M_i) = \max(0, 1 - M_i)$ (зеленая на рисунке выше).

```{note}
Бытует мнение, что метод опорных векторов сегодня нигде не используется из-за его сложности (как минимум квадратичной по числу примеров). Однако, это не так. Линейный SVM вполне неплохо можно применять в задачах с высокой размерностью объектов обучающей выборки, например, для классификации текстов с [Tf-Idf](https://en.wikipedia.org/wiki/Tf–idf) или любым другим разреженным представлением. В частности, [Vowpal Wabbit](https://vowpalwabbit.org/) -- очень эффективная утилита для решения многих задачах машинного обучения -- по умолчанию использует hinge-loss для задач классификации, то есть по сути в этом сценарии применения является линейным SVM. Кусочно-линейная функция потерь хороша тем, что у нее очень простая производная – положительная константа либо ноль. Это удобно использовать с SGD и большими выборками, когда приходится делать миллиарды обновлений весов.

Про прелести Vowpal Wabbit и обучение на гигабайтах данных за считанные минуты можно почитать в [статье](https://habr.com/ru/company/ods/blog/326418/) открытого курса машинного обучения.
```

## Решение задачи метода опорных векторов

Итак, метод опорных векторов сводится к решению задачи оптимизации

$$
    \left\{
        \begin{aligned}
            & \frac{1}{2} \|w\|^2 + C \sum_{i = 1}^{\ell} \xi_i \to \min_{w, b, \xi} \\
            & y_i \left(
                \langle w, x_i \rangle + b
            \right) \geq 1 - \xi_i, \quad i = 1, \dots, \ell, \\
            & \xi_i \geq 0, \quad i = 1, \dots, \ell.
        \end{aligned}
    \right.
$$

Для решения таких _условных_ задач оптимизации с условиями в виде неравенств или равенств часто используют лагранжиан и двойственную задачу оптимизации. Этот подход исчерпывающе описан в классической книге Бойда по оптимизации {cite}`Boyd2006`, а на русском языке можно обратиться к [конспекту](http://www.machinelearning.ru/wiki/images/f/fe/Sem6_linear.pdf) Евгения Соколова. Также для понимания материала рекомендуется рассмотреть ["игрушечный" пример](https://github.com/esokolov/ml-course-hse/blob/master/2016-spring/seminars/sem16-svm.pdf) решения задачи метода опорных векторов в случае линейно-разделимой выборки из 5 примеров.

Построим двойственную задачу к задаче метода опорных векторов.
Запишем лагранжиан:

$$
    L(w, b, \xi, \lambda, \mu)
    =
    \frac{1}{2} \|w\|^2 + C \sum_{i = 1}^{\ell} \xi_i
    -
    \sum_{i = 1}^{\ell} \lambda_i \left[
        y_i \left(
                \langle w, x_i \rangle + b
            \right) - 1 + \xi_i
    \right]
    -
    \sum_{i = 1}^{\ell}
        \mu_i \xi_i.
$$

Выпишем условия Куна-Таккера:

```{math}
:label: KuckTuckerCond1

\nabla_w L = w - \sum_{i = 1}^{\ell} \lambda_i y_i x_i = 0
  \quad\Longrightarrow\quad
    w = \sum_{i = 1}^{\ell} \lambda_i y_i x_i
```

```{math}
:label: KuckTuckerCond2

\nabla_b L = - \sum_{i = 1}^{\ell} \lambda_i y_i = 0
    \quad\Longrightarrow\quad
    \sum_{i = 1}^{\ell} \lambda_i y_i = 0
```

```{math}
:label: KuckTuckerCond3

  \nabla_{\xi_i} L = C - \lambda_i - \mu_i
    \quad\Longrightarrow\quad
    \lambda_i + \mu_i = C
```

```{math}
:label: KuckTuckerCond4

\lambda_i \left[
        y_i \left(
                \langle w, x_i \rangle + b
            \right) - 1 + \xi_i
        \right] = 0
    \quad\Longrightarrow\quad
    (\lambda_i = 0)
        \ \text{или}\
        \left(
            y_i \left(
                \langle w, x_i \rangle + b
            \right)
            =
            1 - \xi_i
        \right)
```


```{math}
:label: KuckTuckerCond5

\mu_i \xi_i = 0
    \quad\Longrightarrow\quad
    (\mu_i = 0)
        \ \text{или}\
        (\xi_i = 0)
```


```{math}
:label: KuckTuckerCond6

    \xi_i \geq 0, \lambda_i \geq 0, \mu_i \geq 0.
```

Проанализируем полученные условия. Из {eq}`KuckTuckerCond1` следует, что вектор весов, полученный в результате настройки SVM, можно записать как линейную комбинацию объектов, причем веса в этой линейной комбинации можно найти как решение двойственной задачи.

В зависимости от значений $\xi_i$ и $\lambda_i$ объекты $x_i$ разбиваются на три категории:

- $\xi_i = 0$, $\lambda_i = 0$. Такие объекты не влияют решение $w$ (входят в него с нулевым весом $\lambda_i$), правильно классифицируются ($\xi_i = 0$) и лежат вне разделяющей полосы. Объекты этой категории называются _периферийными_.
- $\xi_i = 0$, $0 < \lambda_i < C$. Из условия {eq}`KuckTuckerCond4` следует, что $y_i \left(\langle w, x_i \rangle + b \right) = 1$, то есть объект лежит строго на границе разделяющей полосы. Поскольку $\lambda_i > 0$, объект влияет на решение $w$. Объекты этой категории называются _опорными граничными_.
- $\xi_i > 0$, $\lambda_i = C$. Такие объекты могут лежать внутри разделяющей полосы ($0 < \xi_i < 2$) или выходить за ее пределы ($\xi_i \geq 2$). При этом если $0 < \xi_i < 1$, то объект классифицируется правильно, в противном случае -- неправильно. Объекты этой категории называются _опорными нарушителями_.

Отметим, что варианта $\xi_i > 0$, $\lambda_i < C$ быть не может, поскольку при $\xi_i > 0$ из условия дополняющей нежесткости {eq}`KuckTuckerCond5` следует, что $\mu_i = 0$, и отсюда из уравнения {eq}`KuckTuckerCond3` получаем, что $\lambda_i = C$.

Итак, итоговый классификатор зависит только от объектов, лежащих на границе разделяющей полосы, и от объектов-нарушителей (с $\xi_i > 0$).

Построим двойственную функцию. Для этого подставим выражение {eq}`KuckTuckerCond1` в лагранжиан и воспользуемся уравнениями {eq}`KuckTuckerCond2` и {eq}`KuckTuckerCond3` (данные
три уравнения выполнены для точки минимума лагранжиана при
любых фиксированных $\lambda$ и $\mu$):

$$
\begin{align*}
    L &= \frac{1}{2} \left\|
            \sum_{i = 1}^{\ell}
                \lambda_i y_i x_i
        \right\|^2
        -
        \sum_{i, j = 1}^{\ell}
            \lambda_i \lambda_j y_i y_j \langle x_i, x_j \rangle
        -
        b
        \underbrace{\sum_{i = 1}^{\ell}
            \lambda_i y_i}_{0}
        +
        \sum_{i = 1}^{\ell}
            \lambda_i
        +
        \sum_{i = 1}^{\ell}
            \xi_i \underbrace{(C - \lambda_i - \mu_i)}_{0} \\
    &=
    \sum_{i = 1}^{\ell}
        \lambda_i
    -
    \frac{1}{2} \sum_{i, j = 1}^{\ell}
        \lambda_i \lambda_j y_i y_j \langle x_i, x_j \rangle.
\end{align*}
$$

Мы должны потребовать выполнения условий {eq}`KuckTuckerCond2` и {eq}`KuckTuckerCond3` (если они не выполнены, то двойственная функция обращается в минус бесконечность), а также неотрицательность двойственных переменных $\lambda_i \geq 0$, $\mu_i \geq 0$. Ограничение на $\mu_i$ и условие {eq}`KuckTuckerCond3`, можно объединить, получив $\lambda_i \leq C$. Приходим к следующей двойственной задаче:

```{math}
:label: svmDual


    \left\{
        \begin{aligned}
            & \sum_{i = 1}^{\ell}
                \lambda_i
            -
            \frac{1}{2} \sum_{i, j = 1}^{\ell}
                \lambda_i \lambda_j y_i y_j \langle x_i, x_j \rangle
            \to \max_{\lambda} \\
            & 0 \leq \lambda_i \leq C, \quad i = 1, \dots, \ell, \\
            & \sum_{i = 1}^{\ell} \lambda_i y_i = 0.
        \end{aligned}
    \right.
```

Она также является вогнутой, квадратичной и имеет единственный максимум.

## Ядерный переход

Двойственная задача SVM {eq}`svmDual` зависит только от скалярных произведений объектов -- отдельные признаковые описания никак не входят в неё.

```{note}
Обратите внимание, как много это значит: решение SVM зависит только от скалярных произведений объектов (то есть _похожести_, если упрощать), но не от их признаковых описаний объектов. Это значит, что метод обобщается и на те случаи, когда признаковых описаний объектов нет или их получить очень дорого, но зато есть способ задать расстояние (то есть "измерить сходство") между объектами.
```


Значит, можно сделать ядерный переход:

$$
\begin{equation}
    \left\{
        \begin{aligned}
            & \sum_{i = 1}^{\ell}
                \lambda_i
            -
            \frac{1}{2} \sum_{i, j = 1}^{\ell}
                \lambda_i \lambda_j y_i y_j K(x_i, x_j)
            \to \max_{\lambda} \\
            & 0 \leq \lambda_i \leq C, \quad i = 1, \dots, \ell, \\
            & \sum_{i = 1}^{\ell} \lambda_i y_i = 0.
        \end{aligned}
    \right.
\end{equation}
$$

Здесь $K(x_i, x_j)$ -- это функция-ядро, определенная на парах векторов, которая должна быть симметричной и неотрицательно определенной ([теорема Мерсера](http://www.machinelearning.ru/wiki/index.php?title=Теорема_Мерсера)).

Вернемся к тому, какое представление классификатора дает двойственная задача. Из уравнения {eq}`KuckTuckerCond1` следует, что вектор весов $w$ можно представить как линейную комбинацию объектов из обучающей выборки. Подставляя это представление $w$ в классификатор, получаем

$$
\begin{equation}
\label{eq:svmDualClassifier}
    a(x) = \text{sign} \left(
        \sum_{i = 1}^{\ell} \lambda_i y_i \langle x_i, x \rangle + b
    \right).
\end{equation}
$$

Таким образом, классификатор измеряет сходство нового объекта с объектами из обучающей выборки, вычисляя скалярное произведение между ними. Это выражение также зависит только от скалярных произведений, поэтому в нём тоже можно перейти к ядру.

```{note}
Опять подчеркнем, что классификация нового примера зависит только от скалярных произведений -- "похожести" нового примера на примеры из обучающей выборки, и то не все, а только опорные.
```

```{note}
В указанном выше представлении фигурирует переменная сдвига $b$, которая не находится непосредственно в двойственной задаче. Однако ее легко восстановить по любому граничному опорному объекту $x_i$, для которого выполнено $\xi_i = 0, 0 < \lambda_i < C$. Для него выполнено $y_i \left(\langle w, x_i \rangle + b \right) = 1$, откуда получаем

$$
    b = y_i - \langle w, x_i \rangle.
$$

Как правило, для численной устойчивости берут медиану данной величины по
всем граничным опорным объектам:

$$
    b = med \ y_i - \langle w, x_i \rangle, \xi_i = 0, 0 < \lambda_i < C.
$$
```

{numref}`KernelTrick` -- пожалуй, самый известный рисунок в контексте SVM, он иллюстрирует ядерный трюк, в свою очередь, одну из самых красивых идей в истории машинного обучения. За счет ядерного перехода можно достигнуть линейной разделимости выборки даже в том случае, когда исходная обучающая выборка не является линейно разделимой.

```{figure} /_static/qsvmblock/kernel_trick_idea.png
:width: 600px
:name: KernelTrick

Пример разделимости в новом пространстве
```

Наиболее часто используемые ядра:

- _Линейное_ $K(x, y) = \langle x , y \rangle$ -- по сути, линейный SVM, рассмотренный выше;
- _Полиномиальное ядро_ $K(x, y) = (\langle x , y \rangle + c)^d$, определенное для степени ядра $d$ и параметра нормализации $c$;
- _Гауссово ядро_, также известное как RBF (radial-basis functions) $K(x, y) = e^{-\frac{||x - y||^2}{\sigma}}$ c параметром ядра $\sigma$.


## Плюсы и минусы SVM

Плюсы:

- хорошо изучены, есть важные теоретические результаты;
- красиво формулируется как задача оптимизации;
- линейный SVM быстрый, может работать на очень больших выборках;
- линейный SVM так же хорошо интерпретируется, как и прочие линейные модели;
- решение зависит только от скалярных произведений векторов, а идея "ядерного трюка" -- одно из самых красивых в истории машинного обучения;
- нелинейный SVM обобщается на работу с самыми разными типами данных (последовательности, графы и т.д.) за счет специфичных ядер.


Минусы:

- нелинейный SVM имеет высокую вычислительную сложность и принципиально плохо масштабируется (оптимизационную задачу нельзя "решить на подвыборках" и как-то объединить решения);
- нелинейный SVM по сути не интерпретируется ("black box");
- в задачах классификации часто хочется выдать вероятность отнесения к классу, SVM это не умеет делать, а эвристики, как правило, приводят к плохо откалиброванным вероятностям;
- ядерный SVM уступает специфичным нейронным сетям уже во многих задачах, например, в в приложениях к графам
