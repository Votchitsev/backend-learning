# `process.nextTick()` API

Первоисточник: [`process.nextTick(callback[, ...args])`](https://nodejs.org/api/process.html#processnexttickcallback-args). Версия справочника: [Node.js v24.21.0 LTS](https://nodejs.org/docs/latest-v24.x/api/process.html#processnexttickcallback-args).

## Контракт `process.nextTick(callback[, ...args])`

Функция помещает `callback` в next tick queue и передаёт ему дополнительные `args` при вызове. Node.js обрабатывает очередь после завершения текущего JavaScript call stack, до продолжения Event Loop. Это откладывает исполнение, но не запускает callback в другом thread.

```js
const { nextTick } = require('node:process');
nextTick((value) => console.log(value), 'позже');
console.log('сейчас');
// сейчас, позже
```

## Очередь, рекурсия и starvation

Callbacks, добавленные из next tick callback, также попадают в очередь. Пока очередь пополняется, переход к I/O и timers откладывается. Поэтому бесконечная рекурсия может вызвать starvation; даже конечная длинная цепочка увеличивает latency. Это не таймер и не обещание «одного оборота Event Loop».

## Статус API и совместимость

Документация помечает `process.nextTick()` как Legacy и советует для большинства пользовательских сценариев `queueMicrotask()`. Старый API остаётся значимым для совместимости и случаев, когда нужен именно его порядок исполнения. Разница с Promise microtasks зависит от контекста: в CommonJS next ticks обычно идут раньше microtasks; в ESM top-level код уже выполняется в microtask, поэтому порядок может измениться. Подробности: [сравнение](queuemicrotask-vs-nexttick.md).

## Самопроверка

1. Что получит callback из аргументов после `callback`?
2. Почему рекурсивный `nextTick()` способен задерживать I/O?
3. Что означает статус Legacy для выбора API в новом коде?
