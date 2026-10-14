# libuv Design overview

Первоисточник: [libuv Design Overview](https://docs.libuv.org/en/v1.x/design.html). Сверено 14.10.2026 по документации ветки v1.x; актуальный выпуск ветки на дату сверки — [libuv v1.53.0](https://github.com/libuv/libuv/releases/tag/v1.53.0).

## Назначение libuv

libuv — кроссплатформенная библиотека для событийного асинхронного I/O. Она предоставляет не только poller для разных ОС, но и handles, streams, файловые операции, управление threads и другие низкоуровневые средства. Node.js использует libuv, однако JavaScript API Node.js и C API libuv находятся на разных уровнях.

## Handles and requests

**Handle** представляет долгоживущий источник событий: например, TCP server или prepare handle. **Request** описывает обычно короткую операцию: запись в stream либо отдельный DNS запрос. Инициализированный handle закрывают через `uv_close()`; освобождать его память допустимо после close callback. Request не закрывают как handle: он завершается либо отменяется, хотя для `uv_fs_t` нужен `uv_fs_req_cleanup()`, а для `uv_getaddrinfo_t` — `uv_freeaddrinfo()`. `uv_tcp_t` — handle, `uv_write_t` — request. Не все handles имеют одинаковые start/stop функции; `uv_spawn()` одновременно создаёт и запускает процесс.

## The I/O loop

Один loop привязан к одному thread; разные loops могут работать в разных threads. API loop и handles по умолчанию не thread-safe. Сетевые sockets обслуживаются неблокирующими механизмами ОС: epoll, kqueue, event ports или IOCP. Цикл узнаёт о готовности и вызывает callbacks в своём thread.

Проход loop включает обновление времени, проверку жизнеспособности loop, pending/idle/prepare callbacks, расчёт срока ожидания, poll, check, close callbacks и timers. При `UV_RUN_NOWAIT`, остановке loop, закрывающихся handles или отсутствии активной работы ожидание I/O не блокирует thread; иначе срок задаёт ближайший timer, а при отсутствии timers ожидание может быть неограниченным. `UV_RUN_DEFAULT` продолжает проходы, пока loop жив; `UV_RUN_ONCE` и `UV_RUN_NOWAIT` возвращают управление после прохода. Схема libuv описывает его C API и не заменяет объяснение порядка JavaScript microtasks в Node.js.

## File I/O

Для обычных файлов libuv использует блокирующие системные операции внутри общего Thread Pool, потому что переносимого неблокирующего файлового API с нужной семантикой нет. Пул также обслуживает `getaddrinfo`, `getnameinfo` и явно поставленную через `uv_queue_work()` работу. Завершение задачи возвращается в loop; размеры пула ограничены. Сетевой I/O обычно остаётся в thread своего loop и не требует worker на каждое соединение.

## Самопроверка

1. Чем handle отличается от request и как различается их очистка?
2. Что определяет длительность ожидания в poll?
3. Почему libuv применяет разные механизмы к сети и обычным файлам?
