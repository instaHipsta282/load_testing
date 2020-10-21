# Timers

Элементы JMeter из категории `timer` позволяют управлять задержками во время теста.

## Constant Timer
`Timer`, которй позволяет установить точное время задержки в миллисекундах, которое не меняется во время всего теста.

Параметры: 
* Thread Delay (in milliseconds) - точное время задержки.

## Uniform Random Timer
`Timer`, который позволяет установить псевдослучайное время задержки в миллисекундах, при каждом обращении к таймеру
время задержки рассчитывается заново.

Задержка рассчитывается по формуле

`X * RandomDelayMaximum + ConstantDelayOffset = thinkTime.`

Где thinkTime - время задержки а X - псевдослучайное число от 0.0 до 0.9.

Параметры:
* Random Delay Maximum (in milliseconds) - часть задержки, которая будет умножена на число в диапазоне от 0.0 до 0.9.
* Constant Delay Offset (in milliseconds) - постоянное число, которое всегда будет прибавляться к RandomDelayMaximum, 
умноженному на случайное число.

## Precise Throughput Timer

Precise Throughput Timer — это таймер, который позволяет пользователю задать нагрузку (количество сэмплов в 
секунду/минуту/час/и тд.), с которой он хочет запустить свои тесты. Данный таймер, в отличие от 
Constant Throughput Timer, позволяет пользователю более гибко настроить распределение сэмплов по времени. 
Кроме того, выполнение запланировано случайным образом, что позволяет создавать постоянную нагрузку. 
В дополнение к вышесказанному, данный таймер использует пуассоновский процесс для распределения пауз между запросами,
что делает прогон тестов наиболее похожим на действия реального пользователя.
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

## Gaussian Random Timer

Работает аналогично Uniform Random Timer, но случайное число вычисляется по страшной формуле.

Параметры:
* Deviation (in milliseconds)
* Constant Delay Offset (in milliseconds) - постоянное число, которое всегда будет прибавляться к RandomDelayMaximum, 

## Poisson Random Timer

Работает аналогично Uniform Random Timer и Gaussian Random Timer, но использует случайное число из распределения
Пуассона

Параметры:
* Lambda (in milliseconds)
* Constant Delay Offset (in milliseconds) - постоянное число, которое всегда будет прибавляться к RandomDelayMaximum, 

## Synchronizing Timer 

Таймер блокирует выполнение скрипта для потоков до тех пор, пока до него не дайдет количество потоков, указанное
в первом параметре, либо же не закончится время из второго параметра.

Параметры:
* Number of Simulated Users to Group by
* Timeout in milliseconds

## jp@gc - Throughput Shaping Timer

Этот таймер позволяет настроить нужный RPS для теста. Используется в паре с Concurrency Thread Group.