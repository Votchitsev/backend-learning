# libuv Thread pool work scheduling

Первоисточник: [libuv Thread Pool](https://docs.libuv.org/en/v1.x/threadpool.html). Сверено 14.10.2026 по документации ветки v1.x; актуальный выпуск ветки на дату сверки — [libuv v1.53.0](https://github.com/libuv/libuv/releases/tag/v1.53.0).

## Общий Thread Pool

libuv предоставляет общий для всех event loops пул threads. Им пользуются файловые операции, `getaddrinfo`/`getnameinfo` и задачи, переданные через `uv_queue_work()`. Размер по умолчанию — 4; `UV_THREADPOOL_SIZE` задают при старте процесса, верхний предел — 1024. При использовании пула libuv создаёт указанное число threads. Больше threads может повысить пропускную способность независимых задач, но расходует память и не обещает линейного ускорения. Начиная с libuv 1.45 у workers стек 8 MB, с 1.50 имя `libuv-worker`.

Общий пул не делает все его функции thread-safe. Нельзя обращаться к loop и handles из произвольного worker без соответствующего контракта API. Занятые workers задерживают новые задачи из общей очереди.

## Data types / Public members

`uv_work_t` — request задачи. `uv_work_cb(uv_work_t *req)` исполняется в thread пула; `uv_after_work_cb(uv_work_t *req, int status)` исполняется после завершения в thread loop. При успешной отмене через `uv_cancel()` второй callback получает `UV_ECANCELED`. Публичное поле `uv_work_t.loop` указывает на loop, которому принадлежит request, и доступно только для чтения. Общие члены `uv_req_t` также применимы.

## API: `uv_queue_work()`

`uv_queue_work(loop, req, work_cb, after_work_cb)` ставит работу в пул и планирует уведомление в исходном loop. В Node.js это низкоуровневый C API libuv, а не способ автоматически перенести произвольную JavaScript функцию в пул. Отмена request возможна через `uv_cancel()`, но она не означает принудительного прерывания уже выполняющейся системной операции.

## Самопроверка

1. Где выполняются `work_cb` и `after_work_cb`?
2. Почему увеличение `UV_THREADPOOL_SIZE` может не ускорить приложение?
3. Почему общий Thread Pool не означает thread safety всех API libuv?
