# `node:perf_hooks`: измерение производительности

Первоисточник: [`node:perf_hooks`](https://nodejs.org/api/perf_hooks.html). Разбор основан на [Node.js v24.21.0 LTS](https://nodejs.org/docs/latest-v24.x/api/perf_hooks.html). API этой страницы включает совместимую с Web Performance часть и расширения Node.js. Названия публичных элементов ниже сохранены для поиска в оригинале; однотипные свойства сгруппированы по смыслу.

## С чего начать

`performance.now()` измеряет прошедшее время от начала текущего процесса в миллисекундах с высокой точностью. `mark()` ставит именованную отметку, `measure()` создаёт интервал, `PerformanceObserver` получает записи. Измерение времени функции, загруженности Event Loop и задержки Event Loop отвечают на разные вопросы: ни один показатель не описывает производительность полностью.

```js
const { performance } = require('node:perf_hooks');
performance.mark('start');
for (let i = 0; i < 100_000; i++) Math.sqrt(i);
performance.mark('end');
performance.measure('calculation', 'start', 'end');
console.log(performance.getEntriesByName('calculation', 'measure')[0].duration);
```

Результат зависит от машины и нагрузки; не следует превращать одно число в обещание производительности.

## `perf_hooks.performance`

Этот объект хранит performance timeline текущего процесса. `performance.now()` возвращает отметку относительно `performance.timeOrigin`; абсолютная временная основа нужна для сопоставления с другими временными шкалами. `performance.nodeTiming` открывает сведения о старте Node.js, а `performance.toJSON()` сериализует доступные временные сведения.

### Отметки и измерения

- `performance.mark(name[, options])` создаёт `PerformanceMark`; options позволяют задать `detail` и время начала.
- `performance.measure(name[, startMarkOrOptions[, endMark]])` создаёт `PerformanceMeasure` между отметками либо заданными точками времени. Измерение без границ использует доступные значения по умолчанию.
- `performance.getEntries()`, `performance.getEntriesByName(name[, type])`, `performance.getEntriesByType(type)` читают записи всей timeline либо отбирают их по имени и типу.
- `performance.clearMarks([name])`, `performance.clearMeasures([name])`, `performance.clearResourceTimings([name])` удаляют соответствующие записи; без имени очищают всю группу. Это помогает не накапливать измерения бесконечно.

### Ресурсы, функции и Event Loop

- `performance.markResourceTiming(...)` добавляет запись о ресурсе с переданными стадиями загрузки; это специализированный API для инструментов, которые уже знают timingInfo.
- `performance.setResourceTimingBufferSize(maxSize)` ограничивает число записей ресурсов. Event `'resourcetimingbufferfull'` сообщает, что буфер заполнен и записи нужно обработать либо очистить.
- `performance.timerify(fn[, options])` оборачивает функцию так, чтобы её вызовы создавали записи типа `function`; для наблюдения этих записей используют `PerformanceObserver`. Важно измерять то, что реально выполняется внутри обёртки.
- `performance.eventLoopUtilization([utilization1[, utilization2]])` возвращает active, idle и utilization для Event Loop, а переданные предыдущие снимки позволяют получить разницу за интервал. Высокая utilization сама по себе не показывает, какой callback виноват.

## Классы Performance Timeline

### `PerformanceEntry`, `PerformanceMark`, `PerformanceMeasure`

`PerformanceEntry` — общая запись. Её `name` называет измерение, `entryType` определяет тип, `startTime` задаёт начало относительно time origin, `duration` — длительность в миллисекундах. `PerformanceMark` добавляет `detail` к отметке; `PerformanceMeasure` также имеет `detail` для пользовательских сведений об интервале. Объекты записи описывают уже выполненное измерение, а не запускают работу.

### `PerformanceNodeEntry`

Этот тип расширяет общую запись для метрик Node.js. `performanceNodeEntry.detail` содержит данные конкретного типа, `flags` — дополнительные флаги, `kind` — разновидность события, где она применима. Содержимое нельзя интерпретировать одинаково для всех типов.

- **Garbage Collection (`gc`) Details** описывает событие GC, включая его вид и дополнительные флаги.
- **HTTP (`http`) Details** и **HTTP/2 (`http2`) Details** добавляют сведения о соответствующей серверной операции.
- **Timerify (`function`) Details** относится к измеренному вызову функции.
- **Net (`net`) Details** описывает сетевое соединение.
- **DNS (`dns`) Details** описывает DNS работу.

### `PerformanceNodeTiming`

`performance.nodeTiming` описывает этапы запуска и завершения процесса. `nodeStart` и `v8Start` относятся к раннему старту Node.js и V8; `environment` и `bootstrapComplete` — к подготовке окружения и окончанию bootstrap; `loopStart` и `loopExit` — к работе Event Loop. `idleTime` суммирует время ожидания I/O в loop и не равно «свободному CPU всего компьютера». `uvMetricsInfo` даёт счётчики libuv; их смысл следует читать вместе с версией Node.js. Эти свойства нужны для диагностики процесса, а не для измерения отдельного запроса.

### `PerformanceResourceTiming`

Запись описывает стадии загрузки ресурса: `workerStart`, `redirectStart`, `redirectEnd`, `fetchStart`, `domainLookupStart`, `domainLookupEnd`, `connectStart`, `connectEnd`, `secureConnectionStart`, `requestStart`, `responseEnd`. Отсутствующая стадия не обязана означать ошибку: например, перенаправления или TLS могли не использоваться. `transferSize`, `encodedBodySize`, `decodedBodySize` описывают объёмы передачи и тела. `toJSON()` сериализует запись. Сравнивать стадии следует в рамках одной записи и с учётом кэша и транспорта.

## `PerformanceObserver` и `PerformanceObserverEntryList`

`PerformanceObserver.supportedEntryTypes` перечисляет типы записей, которые эта версия Node.js умеет наблюдать. `new PerformanceObserver(callback)` создаёт наблюдатель; `performanceObserver.observe(options)` подписывает его на тип либо набор типов, а `disconnect()` прекращает наблюдение. `takeRecords()` забирает накопленные записи без ожидания callback. Наблюдение стоит отключать, когда измерение закончено, чтобы не расходовать ресурсы.

Callback получает `PerformanceObserverEntryList`. Его `getEntries()`, `getEntriesByName(name[, type])`, `getEntriesByType(type)` возвращают все записи текущего пакета или отфильтрованную часть. Это не обязательно вся глобальная timeline: для неё служат методы `performance.getEntries*()`.

## Event Loop и функции верхнего уровня

- `perf_hooks.createHistogram([options])` создаёт записываемую гистограмму значений в наносекундах. Она полезна для распределения latency, когда среднего недостаточно.
- `perf_hooks.eventLoopUtilization([utilization1[, utilization2]])` — доступный в v24.21.0 псевдоним метода `performance.eventLoopUtilization()`.
- `perf_hooks.monitorEventLoopDelay([options])` создаёт `ELDHistogram`, измеряющую задержку срабатывания цикла относительно ожидаемого времени; наблюдение начинается после `enable()` и останавливается `disable()`. Задержка и utilization измеряют разные стороны поведения.
- `perf_hooks.timerify(fn[, options])` оборачивает функцию для записей типа `function`, аналогично `performance.timerify()`.

## Гистограммы

### `Histogram`

Гистограмма описывает распределение записанных значений. `count`/`countBigInt` дают число наблюдений; `min`/`minBigInt`, `max`/`maxBigInt`, `mean`, `stddev`, `skewness`, `kurtosis` показывают границы и форму распределения. `exceeds`/`exceedsBigInt` учитывают значения вне поддерживаемого диапазона. `percentile(percentile)`, `percentileBigInt(percentile)`, `percentiles`, `percentilesBigInt`, `percentilesAt(percentiles)` отвечают на вопросы о порогах. `reset()` сбрасывает накопленное состояние.

Для анализа формы доступны `cdf(value)`, `ccdf(value)`, `countAt(value)`, `linearBuckets(stepSize)`, `logBuckets(firstBucket, base)`. Для сравнения распределений служат `cliffsD(other)`, `cohensD(other)`, `ksTest(other)`, `mannWhitneyTest(other)`, `welchTest(other)`; они отвечают на разные статистические вопросы, поэтому результат требует подходящей гипотезы и достаточного числа наблюдений. `ewmaMean`, `ewmaStddev`, `ewmaErrorRate` и `burnRate(sloTarget)` помогают отслеживать изменения и нарушения SLO во времени. `percentileCI(percentile[, options])` добавляет доверительный интервал к процентилю. Эти функции не делают маленькую или смещённую выборку репрезентативной.

### `ELDHistogram extends Histogram`

`histogram.enable()` включает измерение Event Loop delay, `histogram.disable()` выключает его, `histogram[Symbol.dispose]()` освобождает ресурс в синтаксисе явного управления ресурсами. Раздел **Cloning an `ELDHistogram`** описывает создание снимка данных для дальнейшего анализа без потери исходного измерения. Значения задержки гистограммы выражены в наносекундах; перед выводом в миллисекундах делите на `1e6`.

### `RecordableHistogram extends Histogram`

`histogram.record(val)` добавляет значение, `recordDelta()` добавляет разницу с прошлой временной точкой, `recordCorrected(val, expectedInterval)` учитывает пропущенные интервалы. `add(other)` и `subtract(other)` объединяют либо вычитают наблюдения, когда это допускают данные. `createHistogram()` создаёт такой записываемый объект; отдельный снимок нужен, если требуется зафиксировать состояние распределения.

## Histogram analysis examples

Раздел **Distribution shape analysis** учит смотреть на форму распределения, а не только на среднее. **SLO monitoring with CDF** и **SLO burn rate monitoring** соотносят задержки с целевым порогом. **Regression detection with KS test** сравнивает распределения до и после изменения. **Batch percentile queries** берёт несколько процентилей за один проход. **Snapshot diffing with subtract** выделяет новые наблюдения между снимками. **Benchmark comparison with Welch's t-test** и **Effect size with Cliff's delta** различают статистическую значимость и величину эффекта. Все примеры требуют одинаковых единиц измерения и сопоставимых условий нагрузки.

## Examples

**Measuring the duration of async operations**: ставьте отметку до операции и измеряйте интервал после её завершения; время ожидания входит в duration. **Measuring how long it takes to load dependencies**: отметки вокруг `require()` показывают стоимость загрузки в этом процессе. **Measuring how long one HTTP round-trip takes**, **Measuring how long the `net.connect` (only for TCP) takes when the connection is successful** и **Measuring how long the DNS takes when the request is successful** показывают, как наблюдать этапы сетевой работы. Их результаты зависят от кэша, сети и условий эксперимента; отдельный образец не доказывает причину задержки.

## Самопроверка

1. Чем измерение `performance.measure()` отличается от Event Loop delay?
2. Почему при анализе latency полезен percentile, а не только mean?
3. Что представляет `PerformanceObserverEntryList`: всю timeline или пакет наблюдателя?
4. В каких единицах измеряется `monitorEventLoopDelay()` и что нужно сделать перед выводом миллисекунд?
