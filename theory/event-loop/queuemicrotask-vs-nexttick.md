# `queueMicrotask()` vs `process.nextTick()`

Первоисточник: [When to use `queueMicrotask()` vs. `process.nextTick()`](https://nodejs.org/api/process.html#when-to-use-queuemicrotask-vs-processnexttick). Версия справочника: [Node.js v24.21.0 LTS](https://nodejs.org/docs/latest-v24.x/api/process.html#when-to-use-queuemicrotask-vs-processnexttick).

## Две очереди

`queueMicrotask(callback)` добавляет callback в очередь microtasks V8; там же выполняются реакции Promise. `process.nextTick(callback[, ...args])` использует отдельную очередь Node.js. В обычном CommonJS-контексте после завершения текущего call stack Node.js сначала обрабатывает next ticks, затем microtasks. В ES modules top-level код уже исполняется как microtask, поэтому microtask, добавленная там, может сработать раньше next tick. Нельзя объявлять один порядок универсальным для всех контекстов.

## Когда что использовать

Официальная рекомендация для большинства пользовательских случаев — `queueMicrotask()`: это стандартный API, доступный и в браузере, и в Node.js, с привычным порядком относительно Promise. `process.nextTick()` сохраняется для совместимости и специфического порядка Node.js. Дополнительные аргументы `nextTick` передаёт прямо в callback; для `queueMicrotask()` потребуется замыкание или `bind`.

```js
queueMicrotask(() => console.log('microtask'));
process.nextTick(() => console.log('next tick'));
```

Этот пример показывает две очереди, но ожидаемый порядок надо описывать вместе с типом модуля и точкой планирования. Ни один механизм не освобождает Event Loop во время тяжёлого синхронного callback. Бесконтрольная цепочка в обеих очередях задерживает другие события.

## Самопроверка

1. В какую очередь попадает callback каждого API?
2. Почему порядок top-level CommonJS и ESM может различаться?
3. Какой API следует предпочесть в новом переносимом коде и почему?
