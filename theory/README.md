# Теория Node.js

Теоретические документы рассчитаны на первое изучение. Каждый из них разбирает один первоисточник; английские названия API сохранены для сверки с официальной документацией. Для повторения можно начать с нужной группы, а при первом чтении пройти порядок ниже.

## Runtime

1. [About Node.js](runtime/about-nodejs.md) — что представляет собой Node.js и почему он подходит для I/O.

## Event Loop

2. [The Node.js Event Loop](event-loop/event-loop.md) — фазы и порядок callbacks.
3. [Understanding `process.nextTick()`](event-loop/understanding-nexttick.md) — короткое введение в next tick queue.
4. [`process.nextTick()` API](event-loop/process-nexttick-api.md) — контракт API и ограничения.
5. [`queueMicrotask()` vs `process.nextTick()`](event-loop/queuemicrotask-vs-nexttick.md) — выбор между механизмами.
6. [Don't Block the Event Loop or the Worker Pool](event-loop/dont-block.md) — последствия долгих задач.

## libuv

7. [Design overview](libuv/design-overview.md) — handles, requests, I/O loop и Worker Pool.
8. [Filesystem](libuv/filesystem.md) — файловые операции и их callbacks.
9. [Thread pool work scheduling](libuv/thread-pool.md) — параметры общего пула.

## Performance

10. [`node:perf_hooks`](performance/perf-hooks.md) — измерения, наблюдатели и статистика.

Материалы дополняют [урок 01](../lessons/lesson-01-nodejs-runtime-event-loop.md). Практическое задание выполняется самостоятельно после изучения документации.
