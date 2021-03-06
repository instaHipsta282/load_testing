# Timers

Элементы JMeter из категории `timer` позволяют управлять задержками во время теста.

## Оглавление
* [Заметки](#Заметки)
* [Constant Timer](#Constant-Timer)
* [Uniform Random Timer](#Uniform-Random-Timer)
* [Gaussian Random Timer](#Gaussian-Random-Timer)
* [Poisson Random Timer](#Poisson-Random-Timer)
* [Precise Throughput Timer](#Precise-Throughput-Timer)
* [Constant Throughput Timer](#Constant-Throughput-Timer)
* [Synchronizing Timer](#Synchronizing-Timer)
* [jp@gc - Throughput Shaping Timer](#jpgc---throughput-shaping-timer)

## Заметки
1. Если таймер находится на одном уровне с сэмплерам и контроллерами, то он действует на все эти сэмплеры, а также на 
сэмплеры в этих контроллерах.
2. Не важно, где стоит таймер (выше, ниже, посередине), он выполняется для всех сэмплеров на этом и нижестоящих уровнях.
3. Таймер выполняется **ДО** запроса
4. Если на запрос влияет два таймера, работают они все.
5. Чтобы увеличить задержку случайного таймера в N раз, нужно умножить на N обе его границы.

## Constant Timer
Таймер, которй позволяет установить точное время задержки в миллисекундах, которое не меняется во время всего теста.

Параметры: 
* Thread Delay (in milliseconds) - точное время задержки.

## Uniform Random Timer
Таймер, который позволяет установить псевдослучайное время задержки в миллисекундах, при каждом обращении к таймеру
время задержки рассчитывается заново.

Задержка рассчитывается по формуле

`X * RandomDelayMaximum + ConstantDelayOffset = thinkTime.`

Где `thinkTime` - время задержки а `X` - псевдослучайное число от 0.0 (включая) до 1.0 (не включая).

Параметры:
* Random Delay Maximum (in milliseconds) - часть задержки, которая будет умножена на число в диапазоне от 0.0 (включая) 
до 1.0 (не включая).
* Constant Delay Offset (in milliseconds) - постоянная часть задержки.

## Gaussian Random Timer
Таймер, который позволяет установить псевдослучайное время задержки в миллисекундах, при каждом обращении к таймеру
время задержки рассчитывается заново.

В этом таймере мы можем задать два числа: константную задержку и смещение. 
В основе этого рандомизатора лежит метод класса Math 
[nextGaussian()](https://docs.oracle.com/javase/8/docs/api/java/util/Random.html#nextGaussian--), где 
среднеквадратическое отклонение равно 1.0.

Время задержки высчитывается слудующим образом:

`(nextGaussian() * смещение) + константная задержка = thinktime`
([sources](https://github.com/apache/jmeter/blob/master/src/components/src/main/java/org/apache/jmeter/timers/GaussianRandomTimer.java))

`nextGaussian()`  
* в 66.26% случаев принимает значение от -среднеквадратическое отклонение (в нашем случае -1.0) до 
+среднеквадратическое отклонение (+1.0).
* В 27.18% случаев это будет -2.0/+2.0.
* В 4.28% -3.0/+3.0.

При константной задержке в 500ms и смещении 100 ms в существенном для нас проценте случаев (68.26%) числа будут в 
диапазоне от 400ms до 600ms `+/- (100 * 1) + 400`, 
еще в 27.18% случаев диапазон будет от 300ms до 700ms `+/- (100 * 2) + 400` 
и вего в 4.28% будет от 200ms до 800ms `+/- (100 * 3) + 400`. 

Все вышесказанное в сумме будет давать ожидаемое нами среднее значение в ~500ms.
В 0.28% значения будут вне пределов этого диапазона 
([Правило трех сигм](https://wiki.loginom.ru/articles/3-sigma-rule.html)).

Параметры:
* Deviation (in milliseconds) - смещение относительно постоянной части задержки.
* Constant Delay Offset (in milliseconds) - постоянная часть задержки. 

## Poisson Random Timer
Таймер, который позволяет установить псевдослучайное время задержки в миллисекундах, при каждом обращении к таймеру
время задержки рассчитывается заново.

Математику постигнуть не смог, данные основанны на синтетических тестах:
* ~99% всех случайных чисел, которые генерирует poisson random timer находятся в диапазоне от 
`constant delay + (0.75 * lambda)` до `constant delay + (1.25 * lambda)`
* Если у вас огромное количество таймеров, большие значения в lambda могут вызвать проблемы. 
Например, при lambda = 100 10 000 000 чисел у меня рассчитывались 3.5 секунды, при lambda = 300 - 7.3 секунды. 
Сильно возрастает нагрузка на cpu.
Параметры:
* Lambda (in milliseconds) - число, от 75 до 125% которого будет прибавляться к постоянной части задержки.
* Constant Delay Offset (in milliseconds) - постоянная часть задержки. 

## Precise Throughput Timer
>Таймер, который позволяет пользователю задать нагрузку (количество сэмплов в секунду/минуту/час/и т.д.), с которой он 
>хочет запустить свои тесты. Данный таймер, в отличие от [Constant Throughput Timer](#Constant-Throughput-Timer), позволяет пользователю более гибко 
>настроить распределение сэмплов по времени. Кроме того, выполнение запланировано случайным образом, что позволяет 
>создавать постоянную нагрузку. В дополнение к вышесказанному, данный таймер использует пуассоновский процесс для 
>распределения пауз между запросами, что делает прогон тестов наиболее похожим на действия реального пользователя.
([Оригинал](https://itnan.ru/post.php?c=1&p=351018)) 

Параметры:
* Target throughput (in samplers per "throughput period")
* Throughput period (seconds)
* Test duration (seconds)

* Number of threads in the batch (threads)
* Delay between threads in the batch (ms)
* Use approximate throughput when sequence length exceeds (samplers)
* Allowed throughput surplus (percents)
* Random seed (change from 0 to random)

## Constant Throughput Timer
Таймер, который позволяет поддерживать постоянную пропускную способность.

Параметры:
* Target throughput (in samplers per minute)
* Calculate throughput based on
    * this thread only
    * all active threads
    * all active threads in current thread group
    * all active threads (shared)     
    * all active threads in current thread group (shared)

## Synchronizing Timer 
Таймер блокирует выполнение скрипта для потоков до тех пор, пока до него не дайдет количество потоков, указанное
в первом параметре, либо же не закончится время из второго параметра.

Параметры:
* Number of Simulated Users to Group by
* Timeout in milliseconds

## jp@gc - Throughput Shaping Timer
Этот таймер позволяет настроить нужный RPS для теста. Используется в паре с Concurrency Thread Group.
