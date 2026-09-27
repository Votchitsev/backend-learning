# Lesson 01 — Node.js Runtime and Event Loop

## 1. Цели урока

После урока ты должен уметь:

- объяснить роли Node.js, V8 и libuv при выполнении программы;
- описать связь call stack, Event Loop, очередей callbacks и microtasks;
- на практическом уровне объяснить фазы Event Loop: timers, pending callbacks,
  poll, check и close callbacks;
- предсказывать порядок выполнения синхронного кода, Promise callbacks,
  `queueMicrotask`, `process.nextTick`, timers и `setImmediate` в типичных
  сценариях;
- различать JavaScript-код на Event Loop, неблокирующий I/O и работу,
  переданную в libuv Worker Pool;
- объяснить, почему CPU-bound синхронная работа задерживает несвязанные
  callbacks и I/O;
- экспериментально проверять гипотезы о поведении runtime, не полагаясь
  только на запомненные правила.

## 2. Предпосылки

Нужны уверенные знания JavaScript/TypeScript: функции, callbacks, Promises,
модули и запуск программ из командной строки.

Это первый урок curriculum, поэтому завершённых предыдущих уроков не
требуется. Согласно `PROGRESS.md`, Lesson 01 ещё не начат; подтверждённых
пробелов, требующих дополнительной подготовки, пока нет.

## 3. Документация

Читай материалы в указанном порядке. Цель — построить рабочую модель runtime,
а не запомнить диаграмму Event Loop дословно.

1. [About Node.js](https://nodejs.org/en/about)
   - Сосредоточься на определениях asynchronous, event-driven runtime и на
     модели обработки множества соединений небольшим числом threads.
   - После чтения объясни своими словами, почему фраза «Node.js однопоточный»
     одновременно полезна и недостаточна.

2. [The Node.js Event Loop](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick)
   - Изучи разделы про фазы Event Loop, `poll`, `check`, timers,
     `setImmediate()` и `process.nextTick()`.
   - Обрати внимание: порядок между timer и `setImmediate()` зависит от
     контекста планирования; это не универсальная гонка с одним ответом.

3. [Understanding process.nextTick()](https://nodejs.org/en/learn/asynchronous-work/understanding-processnexttick)
   и [`process.nextTick()` API](https://nodejs.org/api/process.html#processnexttickcallback-args)
   - Разбери, когда опустошается next tick queue и почему рекурсивный
     `process.nextTick()` может задерживать Event Loop.
   - Прочитай подраздел **When to use `queueMicrotask()` vs.
     `process.nextTick()`**. Учитывай различия порядка в CommonJS и ESM;
     `process.nextTick()` имеет статус Legacy, а для большинства userland
     случаев документация рекомендует `queueMicrotask()`.

4. [Don't Block the Event Loop (or the Worker Pool)](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop)
   - Изучи разделы **A quick review of Node**, **What code runs on the Event
     Loop?**, **What code runs on the Worker Pool?** и **Why should I avoid
     blocking...**.
   - Зафиксируй, какие стандартные Node.js APIs используют Worker Pool и чем
     CPU-bound работа отличается от ожидания I/O.

5. [libuv Design overview](https://docs.libuv.org/en/v1.x/design.html)
   - Нужны разделы **The I/O loop**, **File I/O** и **Thread pool work
     scheduling**.
   - Цель — понять границу между JavaScript, Node.js bindings, операционной
     системой и libuv. Детали реализации отдельных платформ запоминать не
     нужно.

## 4. Ключевые понятия

- Node.js — JavaScript runtime, а не язык и не web framework.
- V8 выполняет JavaScript и управляет microtask queue, используемой Promise
  reactions и `queueMicrotask()`.
- JavaScript callbacks одного Node.js isolate обычно выполняются по одному,
  до завершения текущего callback.
- Event Loop координирует готовые события и выполняет callbacks по фазам.
- Асинхронный API не означает, что его работа обязательно выполняется в
  отдельном thread: часть I/O обслуживается механизмами ОС, часть операций
  использует libuv Worker Pool.
- Worker Pool не делает произвольный синхронный JavaScript неблокирующим.
- Microtasks выполняются между переходами к дальнейшей работе Event Loop;
  бесконтрольное добавление microtasks или next ticks способно вызвать
  starvation.
- Timer задаёт минимальную задержку, после которой callback становится
  допустимым к выполнению, а не точное время его запуска.
- CPU-bound callback удерживает Event Loop до своего завершения и увеличивает
  latency других операций.
- Наблюдаемый порядок может зависеть от версии Node.js, module system и
  контекста, в котором callbacks были запланированы.

## 5. Вопросы для проверки понимания

Ответь письменно до начала практики.

1. Какие части фразы «Node.js выполняет всё в одном thread» неверны или
   вводят в заблуждение?
2. Почему callback готового timer может выполниться заметно позже указанного
   delay?
3. Почему Promise не превращает тяжёлый синхронный расчёт внутри `.then()` в
   фоновую работу?
4. Чем microtask queue отличается от фаз Event Loop на уровне наблюдаемого
   поведения программы?
5. Почему длинная цепочка `process.nextTick()` может задержать timer и I/O?
6. Можно ли всегда заранее утверждать, что `setTimeout(fn, 0)` выполнится до
   `setImmediate(fn)`? От какого контекста зависит ответ?
7. Что происходит между вызовом асинхронного filesystem API и запуском его
   JavaScript callback?
8. Почему использование Worker Pool одной операцией может повлиять на
   latency другой, логически независимой операции?
9. Чем CPU-bound работа принципиально отличается от ожидания сетевого ответа?
10. Какие измерения помогли бы доказать, что Event Loop блокируется, а не
    просто «работает медленно»?

## 6. Практическое задание

### Цель

Экспериментально исследовать порядок выполнения и задержки в Node.js, а затем
объяснить результаты через модель V8, Event Loop и libuv.

### Требования

Создай в `practice/lesson-01-event-loop/` небольшой TypeScript-проект с
несколькими независимо запускаемыми экспериментами и отчётом `README.md`.

Эксперименты должны охватить:

- синхронный код, Promise callback, `queueMicrotask()`, `process.nextTick()`,
  `setTimeout(..., 0)` и `setImmediate()`;
- сравнение планирования из top-level кода и из callback асинхронного I/O;
- измерение фактической задержки timer до и во время CPU-bound синхронной
  работы;
- сравнение CPU-bound JavaScript с одной асинхронной операцией стандартной
  библиотеки, использующей libuv Worker Pool;
- starvation-сценарий с конечным, безопасно ограниченным числом
  `process.nextTick()` или microtasks.

Для каждого эксперимента в `README.md` до первого запуска запиши:

- предполагаемый порядок событий или ожидаемое влияние на latency;
- фактический результат с измерениями, где они применимы;
- объяснение результата через call stack, очереди, фазу Event Loop и/или
  Worker Pool;
- вывод: подтвердилась ли гипотеза и почему.

Также добавь итоговую схему ответственности: что в твоих экспериментах
выполняли V8, Node.js/libuv, Worker Pool и операционная система. Схема может
быть текстовой или Mermaid.

### Ограничения

- Используй актуальную LTS-версию Node.js и зафиксируй точную версию в отчёте.
- Используй только встроенные Node.js APIs и уже доступные средства запуска
  TypeScript; сторонние runtime-библиотеки не нужны.
- Не используй `worker_threads`, `child_process`, `cluster` и web frameworks:
  offloading пользовательского JavaScript будет отдельной темой.
- CPU-bound эксперимент должен завершаться за разумное время и не должен
  расходовать неограниченную память.
- Starvation-эксперимент обязан иметь жёсткий предел итераций.
- Не подгоняй предсказание после запуска: изменения гипотезы фиксируй отдельно.
- Не утверждай универсальный порядок там, где документация указывает
  зависимость от контекста, module system или версии runtime.

### Ожидаемое поведение

- Синхронные логи появляются до отложенных callbacks.
- CPU-bound синхронная работа увеличивает наблюдаемую задержку timer и другой
  готовой работы на Event Loop.
- Асинхронная операция может выполняться вне JavaScript call stack, однако её
  завершающий JavaScript callback всё равно ждёт возможности выполниться на
  Event Loop.
- Ограниченная цепочка next ticks или microtasks задерживает переход Event
  Loop к последующим фазам, но программа в итоге завершается.
- Эксперименты с `setTimeout(..., 0)` и `setImmediate()` показывают влияние
  контекста планирования, а не формулируются как одно абсолютное правило.

### Критерии приёмки

- Все эксперименты запускаются отдельными командами и завершаются без ручного
  прерывания.
- В исходниках нет намеренно бесконечных циклов и неограниченной рекурсии.
- До каждого фактического результата в отчёте сохранена проверяемая гипотеза.
- Порядок событий виден по однозначным labels; измерения времени содержат
  единицы измерения.
- Отчёт различает Event Loop и Worker Pool и не описывает любой async API как
  «отдельный thread».
- Объяснение учитывает, что callback завершившейся фоновой операции исполняет
  JavaScript на Event Loop.
- Есть минимум один результат, для которого объяснено влияние контекста
  планирования или module system.
- Итоговый вывод объясняет, почему Node.js хорошо подходит для большого числа
  ожидающих I/O операций и почему CPU-bound JavaScript требует особого
  внимания.

### Разрешённые инструменты и APIs

- `node:process`;
- timers: `setTimeout`, `setImmediate`;
- Promises и `queueMicrotask`;
- `node:fs` или `node:fs/promises` для I/O-контекста;
- один подходящий asynchronous API из `node:crypto` для наблюдения Worker
  Pool;
- `node:perf_hooks` или `process.hrtime.bigint()` для измерений;
- shell-команды для запуска и повторения экспериментов.

## 7. Дополнительное задание

Исследуй влияние размера libuv Worker Pool на несколько конкурентных
операций одного встроенного API. Сначала сформулируй гипотезу, затем сравни
не менее двух запусков с разным `UV_THREADPOOL_SIZE` и объясни:

- что изменилось в общей длительности и времени завершения отдельных задач;
- почему увеличение pool не гарантирует линейного ускорения;
- почему этот эксперимент ничего не меняет в однопоточном выполнении обычных
  JavaScript callbacks на Event Loop.

Это задание необязательно и не требуется для отправки основной практики на
review.
