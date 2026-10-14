# The Node.js Event Loop

Первоисточник: [The Node.js Event Loop: timers, `nextTick`, phases, `setImmediate`](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick). Сверено 14.10.2026; объяснения ориентированы на Node.js v24.21.0 LTS.

## What is the Event Loop?

Event Loop позволяет Node.js координировать неблокирующий I/O, пока JavaScript callbacks одного isolate обычно исполняются последовательно. Сетевые события могут ожидаться средствами ОС. Завершившаяся работа не прерывает текущий callback: она будет обработана, когда выполнение вернётся к циклу.

## Event Loop Explained

После первоначального выполнения скрипта Node.js проходит фазы обработки готовой работы. У каждой фазы есть свой тип callbacks. Диаграмма статьи упрощает детали платформ; её нельзя использовать как абсолютное расписание каждого callback. После Node.js 20 (libuv 1.45) timers выполняются после poll, что особенно важно при сравнении timers и `setImmediate()`.

## Phases Overview

Практическая модель: **timers** обрабатывает достигшие порога `setTimeout` и `setInterval`; **pending callbacks** — некоторые отложенные I/O callbacks; **idle/prepare** — внутренняя работа; **poll** — ожидание и обработка I/O; **check** — `setImmediate`; **close callbacks** — события закрытия. Node.js может завершить процесс, когда нет активной работы, удерживающей цикл.

## Phases in Detail

### pending callbacks

Здесь исполняются некоторые I/O callbacks, отложенные до следующего прохода цикла, например сообщения об отдельных TCP ошибках. Не все I/O callbacks относятся сюда.

### poll

Фаза получает новые I/O события и выполняет связанные callbacks. Если готовой работы нет, цикл может ожидать событий с учётом timers и `setImmediate`. Долгий callback задержит переход к другим событиям, даже если они уже готовы.

### check

Здесь запускаются callbacks, переданные `setImmediate()`. Вызов `setImmediate()` из I/O callback обычно приводит к исполнению в следующей check фазе, после текущей poll работы.

### close callbacks

Обрабатываются некоторые события закрытия, например `socket.on('close', ...)`. Точный момент зависит от способа закрытия: `destroy()` и обычное закрытие могут пройти разными путями.

### timers

Timer задаёт минимальный порог задержки, а не точный момент вызова. Если Event Loop занят, callback с `0` или `1` мс сработает позже. Для Node.js 24 следует учитывать изменение порядка timers относительно poll в libuv 1.45+; схему старых версий нельзя переносить буквально.

## `setImmediate()` vs `setTimeout()`

Из top-level кода порядок `setImmediate(fn)` и `setTimeout(fn, 0)` не гарантирован: он зависит от времени подготовки цикла и других условий. Если оба запланированы внутри I/O callback, `setImmediate` обычно исполнится раньше timer, потому что check следует за poll. Не строите корректность программы на гонке этих двух callbacks.

## `process.nextTick`

### Understanding `process.nextTick()`

Next tick callback исполняется после текущей JavaScript операции, до продолжения Event Loop. Это не отдельная фаза на диаграмме. Подробнее: [отдельный документ](understanding-nexttick.md).

### Why would that be allowed?

Так можно закончить настройку объекта до вызова callback или сообщить ошибку асинхронно, сохранив предсказуемый контракт API. Но рекурсивное пополнение очереди способно не дать I/O получить очередь исполнения.

### `process.nextTick()` vs `setImmediate()`

Next tick выполняется до перехода к дальнейшим фазам; `setImmediate` ждёт check фазу. Если нужно уступить управление I/O, `setImmediate` подходит лучше длинной цепочки next ticks.

### Why use `process.nextTick()`?

В статье показан случай, когда callback нельзя вызвать синхронно до возвращения из функции: вызывающая сторона должна успеть установить обработчики. Для нового пользовательского кода документация API в большинстве случаев рекомендует `queueMicrotask()`; см. [сравнение](queuemicrotask-vs-nexttick.md).

## Самопроверка

1. В какой фазе срабатывает `setImmediate()` и почему timer из top-level не имеет фиксированного порядка относительно него?
2. Что делает poll, когда I/O не готов и есть ожидающие timers?
3. Почему `process.nextTick()` отсутствует среди фаз Event Loop?
4. Почему callback готового timer может опоздать?
