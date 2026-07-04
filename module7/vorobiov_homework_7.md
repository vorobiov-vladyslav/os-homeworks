# Воробйов Владислав homework 7

Обраний варіант: **A — аналіз життєвого циклу контейнера**.

Застосунок — вбудований HTTP-сервер Python (`python -m http.server`).

## 1. Запуск

```bash
docker run -d --name webapp -p 8080:8000 python:3.12-slim python -m http.server 8000
curl -s -o /dev/null -w "HTTP %{http_code}\n" http://localhost:8080/
```

Результат:
```text
a36b1901ee1c5e8b949ff8509d9b541440f9f3759d33f63911a59da9b926e2ba
HTTP 200
```
Контейнер запущено у фоні (`-d`), порт 8000 контейнера прокинуто на 8080 хоста (`-p 8080:8000`). Сервер відповідає `200 OK`.

## 2. Процес: хто є PID 1

Усередині контейнера (мінімальний образ без `ps`, тому використано `/proc` — прямий аналог):
```bash
docker exec webapp sh -c 'tr "\0" " " < /proc/1/cmdline; echo; ls -d /proc/[0-9]*'
```
```text
python -m http.server 8000
/proc/1
/proc/9
```

Той самий процес з боку хоста:
```bash
docker top webapp
```
```text
PID                 USER                TIME                COMMAND
3692828             root                0:00                python -m http.server 8000
```

**Чому саме він PID 1?** Контейнер — це не віртуальна машина, а ізольований процес у власному PID-namespace. Першим (і єдиним) процесом у цьому namespace стає команда з `CMD`/аргументів `docker run` — наш `python`, тому він отримує PID 1. `/proc/9` — це лише тимчасовий `sh` від самого `docker exec`. Життя контейнера дорівнює життю PID 1: коли цей процес завершується — контейнер зупиняється.

## 3. Завершення через `docker stop`

```bash
time docker stop webapp
docker inspect -f 'ExitCode: {{.State.ExitCode}}' webapp
docker ps -a --filter name=webapp --format '{{.Names}}: {{.Status}}'
```

Результат:
```text
docker stop webapp  0.02s user 0.02s system 0% cpu 10.192 total
ExitCode: 137
webapp: Exited (137) Less than a second ago
```

**Який сигнал отримує процес:** `docker stop` надсилає PID 1 сигнал `SIGTERM` — «ввічливе» прохання завершитися.

**Що відбувається, якщо процес його ігнорує:** для PID 1 ядро не застосовує дефолтні обробники сигналів — сигнал діє лише якщо процес сам встановив handler. `python -m http.server` обробник `SIGTERM` не встановлює, тому просто проігнорував сигнал. Docker почекав grace-період (10 секунд за замовчуванням — рівно стільки й тривала команда: `10.192 total`) і надіслав `SIGKILL`, який ігнорувати неможливо. Код виходу `137 = 128 + 9` (номер сигналу `SIGKILL`) підтверджує примусове завершення.

## 4. Логи

```bash
docker logs webapp
```

Результат:
```text
192.168.215.1 - - [04/Jul/2026 06:44:00] "GET / HTTP/1.1" 200 -
192.168.215.1 - - [04/Jul/2026 06:44:00] "GET / HTTP/1.1" 200 -
```

**Звідки вони беруться:** усередині контейнера ніхто логи у файли не пише — це просто `stdout`/`stderr` процесу PID 1. Docker перехоплює ці потоки і зберігає їх через logging-драйвер (за замовчуванням `json-file`, у `/var/lib/docker/containers/<id>/<id>-json.log`), а `docker logs` читає звідти. Тому логи доступні навіть після зупинки контейнера.

## Висновок

Життя контейнера визначає його головний процес (PID 1): контейнер працює, поки працює він. Наш контейнер завершився не сам — `docker stop` надіслав `SIGTERM`, процес без обробника сигналу проігнорував його, і через 10 секунд Docker примусово вбив його `SIGKILL` (exit code 137).
