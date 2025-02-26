(qaoa)=

# Quantum Approximate Optimization Algorithm

## Введение

В лекции рассматривается еще один алгоритм для приближенного решения [$NP$-задач комбинаторный оптимизации](copt), который называется **Q**auntum **A**pproximate **O**ptimization **A**lgorithm (далее **QAOA**) {cite}`farhi2014quantum`.

### Квантовый отжиг (повторение)

В прошлых лекция активно [рассказывалось](dwave) о квантовых _аннилерах_ (_отжигателях_, англ. _annealers_) -- аналоговых устройствах, реализующих поиск основного состояния системы. Выпишем еще раз [гамильтониан](../dwaveblock/dwave.html#id4), который там используется:

$$
\mathcal{H}_{Ising}=\underbrace{{-A(t)}\left(\sum_{i} \hat{\sigma}_{x}^{(i)}\right)}_{\text {Initial Hamiltonian }}+\underbrace{{B(t)}\left(\sum_{i} h_{i} \hat{\sigma}_{z}^{(i)}+\sum_{i,j} J_{i, j} \hat{\sigma}_{z}^{(i)} \hat{\sigma}_{z}^{(j)}\right)}_{\text {Final Hamiltonian }},
$$

В процессе квантового отжига плавно меняются параметры $A$ и $B$:


```{figure} /_static/dwaveblock/dwave/fig_3.png
:name: fig_1
:width: 444px

Пример расписания отжига: функций $A(t)$, $B(t)$.
```

В той же [лекции](swave) о `D-Wave` указывалась главная проблема -- риск перехода системы из основного состояния в возбужденное при недостаточно медленном изменении параметров $A$ и $B$:

```{figure} /_static/dwaveblock/dwave/fig_1.png
:name: fig_2
:width: 444px

Типичная зависимость от времени энергетических уровней гамильтонианов, используемых в квантовом отжиге
```

Таким образом, большой проблемой для нас является выбор правильного "расписания" отжига, то есть зависимостей $A(t)$ и $B(t)$.

### От железа к симуляции

У отжига "в железе" есть существенная проблема -- трудно правильным образом подбирать расписание отжига. Но есть и другая идея! Изначально квантовые компьютеры создавались для симуляции квантовой динамики. Воспользуемся техникой симуляции, которая называется _trotterization_.

### Trotterization и симуляция аннилера

В данном случае наша цель -- получить выражение для финального состояния $\ket{\Psi}$, которое будет отвечать решению квантового отжигателя. Финальное состояние есть решение уравнения Шредингера:

$$
\ket{\Psi(t)} = e^{-i\mathcal{H}_{Ising}(t)t}\ket{\Psi(0)} = e^{-i(A(t) \mathcal{H}_{initial} + B(t)\mathcal{H}_{cost})t}\ket{\Psi(0)},
$$

где $\mathcal{H}_{initial}$ -- это _tunneling_ или начальный гамильтониан, а $\mathcal{H}_{cost}$ -- это так называемый _problem_ или _cost_ гамильтониан, который отвечает задаче. Подробно эти темы обсуждались в [лекции о D-Wave](dwave). Для тех, кто сейчас ничего не понял, рекомендуется также вернуться к лекции [о переходе от комбинаторных задач к гамильтонианам](np2ising), чтобы понять, как получается _cost_ гамильтониан.

```{note}
Уравнение Шредингера практически не решается численно для сколько-нибудь больших задач, поэтому мы примем как факт то, что точно посчитать квантовую динамику аннилера у нас не выйдет, тем более не выйдет как-то пытаться оптимизировать
```

Для решения проблемы "нерешаемости" уравнения Шредингера перейдем от непрерывного времени $t$ и зависимостей $A(t)$ и $B(t)$ к $N$ дискретных моментов времени $t_1, t_2, ..., t_n$. Тогда можно заменить расписание отжига в виде непрерывных коэффициентов на набор дискретных коэффициентов, каждый из которых отвечает своему моменту времени:

$$
& A(t) \to \gamma_1, \gamma_2, ..., \gamma_N \\
& B(t) \to \beta_1, \beta_2, ..., \beta_N
$$

Финальное состояние записывается так:

$$
\ket{\Psi(t)} = e^{-i\gamma_1 \mathcal{H}_{initial}}e^{-i\beta_1\mathcal{H}_{cost}} ... e^{-i\gamma_N \mathcal{H}_{initial}}e^{-i\beta_N\mathcal{H}_{cost}} \ket{\Psi(0)}
$$

### Оптимизация расписания

Теперь задача оптимизации расписания по своей сути сведена к следующей:

$$
& \arg\min_{\gamma_1, ..., \gamma_N, \beta_1, ..., \beta_N} {\braket{\Psi_{final} | \mathcal{H}_{cost} | \Psi_{final}}} \\
& \ket{\Psi_{final}} = e^{-i\gamma_1 \mathcal{H}_{initial}}e^{-i\beta_1\mathcal{H}_{cost}} ... e^{-i\gamma_N \mathcal{H}_{initial}}e^{-i\beta_N\mathcal{H}_{cost}}\ket{\Psi(0)}
$$

А это уже хорошо знакомая задача оптимизации результата измерения состояния, заданного при помощи некоторой [**VQC**](vqc), которая параметризирована набором действительных чисел $\gamma_1, ..., \gamma_N, \beta_1, ..., \beta_N$. И эта задача решается хорошо уже знакомыми [градиентными методами](gradients).

## Пример задачи оптимизации

Рассмотрим задачу оптимизации $n$-разрядного набора данных, для которого нужно найти некоторый минимум (или максимум). Алгоритм задается двумя гамильтонианами $H_{p}$ и $H_{M}$, а также $2p$ параметрами: $\gamma_{1}, ..., \gamma_{p}$ и $\beta_{1}, ..., \beta_{p}$.

`QAOA` использует унитарный оператор $U(\beta,\gamma)$, принимающий на вход вещественные параметры $\beta$, $\gamma$, и описывается уже знакомым квантовым состоянием $\ket{\Psi}$. Цель поиска -- найти те самые оптимальные $\beta_{\text{opt}}$ и $\gamma_{\text{opt}}$.

Оператор $U$ состоит из двух частей:

- оператор, меняющий фазу $U_{\text{phase}}$

    $$
    U_{\text{phase}}(\gamma) = e^{ -i {\gamma} H_{\text{phase}} }
    $$

- оператор, смешивающий кубиты $U_{\text{mixer}}$

    $$
    U_{\text{mixer}}(\beta) = e^{ -i {\beta} H_{\text{mixer}} }
    $$

Оператор $U_{\text{phase}}$ совершает вращение относительно осей $Z$ или $Y$ с помощью соответствующих матриц Паули

$$
H_{\text{phase}} = Z \ or \ Y \ \text{axis rotation}
$$

```{figure} /_static/qaoablock/hamiltonian_u_phase.png
:name: hamiltonian_u_phase
:width: 444px

Оператор $U_{\text{phase}}$
```

$U_{\text{mixed}}$ в классическом случае использует матрицу $XNOT$.

Операторы применяются к начальному состоянию $\ket{\Psi_{0}}$ (путем поочередного применения гамильтонианов $H_{p}$ и $H_{M}$) последовательно $р$ раз (или, иначе говоря, используются $p$ слоев), где продолжительность $j$-й итерации определяется параметрами $\gamma_{j}$ и $\beta_{j}$ соответственно

$$
\ket{\phi(\beta,\gamma)} = \underbrace{U_{\text{mixer}}(\beta) U_{\text{phase}}(\gamma) \ ... \ U_{\text{mixer}}(\beta) U_{\text{phase}}(\gamma)}_{\text {p times}}{\ket{\Psi_0}}
$$

Общая схема для $n$ кубитов выглядит следующим образом

```{figure} /_static/qaoablock/general_scheme_for_n_qubits.png
:name: general_scheme_for_n_qubits
:width: 444px

Общая схема для $n$ кубитов
```

Итак, алгоритм состоит из следующих основных этапов:

1. приготовление начального состояния $\ket{\Psi_{0}}$ из $n$ кубитов. Начальное состояние выбирается как равное состояние суперпозиции всех возможных решений

    $$
    \ket{\Psi_{0}} = \frac{1}{\sqrt{2^n}} \sum_{x} \ket{x}
    $$

2. последующее применение к каждому кубиту матриц Адамара для осуществления суперпозиции всевозможных состояний

    ```{figure} /_static/qaoablock/the_1t_step_alg.png
    :name: the_1t_step_alg
    :width: 222px
    ```
3. применяем оператор вращения фазы

    $$
    U_{\text{phase}} = \sum_{i \neq j}^{n-1} e^{-i \gamma Z_i Z_j}
    $$

    например, вот так

    $$
    H_p = (I_0 \otimes Z_1 \otimes I_2 \otimes Z_3)
    $$

    ```{figure} /_static/qaoablock/the_2d_step_alg.png
    :name: the_2d_step_alg
    :width: 222px
    ```

    Напоминаем, как выглядит данный оператор в матричном виде: $Z = \begin{bmatrix} 1 & 0 \\ 0 & 1\end{bmatrix}$.

4. применяем смешивающий оператор

    $$
    U_{\text{mixer}} = \sum_{i=0}^{n-1} e^{-i \beta X_i}
    $$

    к примеру, так

    $$
    U_{\text{mixer}} = (I \otimes I \otimes X \otimes Z)
    $$

    ```{figure} /_static/qaoablock/the_3d_step_alg.png
    :name: the_3d_step_alg
    :width: 222px
    ```

    $$
    X = \begin{bmatrix} 0 & 1 \\ 1 & 0\end{bmatrix}
    $$

В данном алгоритме используется адиабатический метод эволюции состояния {cite}`farhi2000quantum` $\ket{\Psi_0}$ с переменным гамильтонианом: на каждой итерации параметры $\beta$ и $\gamma$ понемногу изменяются.

Далее производится измерение финального состояния в $Z$-базисе и вычисление $\bra{\Psi(\beta,\gamma)}H_{phase}\ket{\Psi(\beta,\gamma)}$. Найденный минимум будет соответствовать оптимальным $\beta$ и $\gamma$.

Описанные выше шаги могут быть полностью повторены с обновлёнными наборами временных параметров в рамках классического цикла оптимизации (такого как градиентный спуск или другие подходы), используемого для оптимизации параметров алгоритма.

Возвращается лучшее решение, найденное за всё время поиска.
## Quantum Alternating Operator Ansatz

Применение "анзаца" в алгоритме квантовой приближенной оптимизации заключается в модернизации оператора смешивания $U_{mixer}$ и предполагает использование $CNOT$, а не $X$.

Анзац рассматривает более общие параметризированные унитарные трансформации, а не только соответствующие эволюции фиксированного локального гамильтониана во времени. Он позволяет более эффективно реализовывать операции смешивания, особенно в задачах оптимизации с жесткими ограничениями.

На рисунках ниже представлена абстрактная визуализация "смешивания" и обозначение оператора:

```{figure} /_static/qaoablock/ansatz_mixing.png
:name: ansatz_mixing
:width: 444px

"Смешивание"
```

```{figure} /_static/qaoablock/ansatz_operator_designation.png
:name: ansatz_operator_designation
:width: 444px

Обозначение оператора
```

Классически семейство анзацев можно поделить на три основных типа:
- Hardware Efficient Ansatz (HEA) – запутывающий все кубиты;
- Alternating Layered Ansatz (ALT) {cite}`farhi2014quantum`;
- Tensor Product Ansatz (TEN) {cite}`huggins2019towards`.

На рисунке ниже изображены упрощённые (без учета анзаца и фазовых гейтов) для понимания схемы описанных компоновок -- по одному слою каждого типа:

```{figure} /_static/qaoablock/ansats_types.png
:name: ansatz_types
:width: 444px
```

Конечно, к компоновке смешанных гейтов можно подходить сколь угодно творчески, и пример общей схемы, реализующей QAOAz, представлен ниже:

```{figure} /_static/qaoablock/ansatz_sample.png
:name: ansatz_sample
:width: 444px
```
