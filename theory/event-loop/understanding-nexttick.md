# Understanding `process.nextTick()`

Первоисточник: [Understanding `process.nextTick()`](https://nodejs.org/en/learn/asynchronous-work/understanding-processnexttick). Сверено 14.10.2026; поведение объясняется для Node.js v24.21.0 LTS.

## Tick и текущий call stack

Статья называет tick возвращение runtime к JavaScript для обработки события. `process.nextTick(callback)` откладывает callback до завершения текущей операции JavaScript. Callback не запускается посреди текущего call stack и не относится к обычным фазам Event Loop.

```js
console.log('A');
process.nextTick(() => console.log('C'));
console.log('B');
// A, B, C
```

## Когда выполняется callback

После опустошения текущего call stack Node.js обрабатывает очередь next ticks прежде, чем продолжить работу Event Loop и перейти к timers или другим готовым событиям. Поэтому `setTimeout(fn, 0)` не равнозначен `process.nextTick(fn)`: timer допускается к запуску лишь позднее, и его задержка не гарантирует точное время. Формулировку статьи о «следующей итерации» следует понимать как «после текущего JavaScript», а не как отдельную фазу следующего оборота Event Loop.

Длинная цепочка callbacks, каждый из которых снова вызывает `process.nextTick()`, способна задержать I/O и timers. API не следует применять как общий способ сделать тяжёлую работу неблокирующей. Для выбора между ним и `queueMicrotask()` см. [отдельное сравнение](queuemicrotask-vs-nexttick.md).

## Самопроверка

1. Может ли next tick callback прервать выполняющуюся функцию?
2. Почему timer с задержкой 0 не заменяет `process.nextTick()`?
3. Что произойдёт с I/O, если постоянно добавлять новые next ticks?
