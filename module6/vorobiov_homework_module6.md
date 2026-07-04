# Воробйов Владислав homework module 6

Обраний варіант: **A — скрипт бекапу логів** (`backup.sh`).

## Код скрипта

```bash
#!/bin/bash

LOCK_FILE="/tmp/backup.lock"

# Перевірка аргументів: має бути рівно 2 і обидва — існуючі каталоги
if [ "$#" -ne 2 ] || [ ! -d "$1" ] || [ ! -d "$2" ]; then
    echo "Usage: ./backup.sh <log_dir> <backup_dir>"
    exit 1
fi

LOG_DIR="$1"
BACKUP_DIR="$2"

# Захист від паралельного запуску: якщо lock-файл існує — виходимо
if [ -e "$LOCK_FILE" ]; then
    echo "Backup already running"
    exit 0
fi

# Створюємо lock-файл і гарантуємо його видалення при будь-якому завершенні
touch "$LOCK_FILE"
trap "rm -f $LOCK_FILE" EXIT

# Ім'я архіву містить поточну дату і час
ARCHIVE_NAME="logs_backup_$(date +%Y-%m-%d_%H-%M).tar.gz"
ARCHIVE_PATH="$BACKUP_DIR/$ARCHIVE_NAME"

# Архівуємо всі файли з каталогу логів; -C дозволяє не тягнути повний шлях
if tar -czf "$ARCHIVE_PATH" -C "$LOG_DIR" .; then
    echo "Backup created: $(realpath "$ARCHIVE_PATH")"
else
    echo "Backup failed"
    exit 2
fi
```

## Що відбувається

1. Скрипт перевіряє, що передано рівно 2 аргументи і обидва є існуючими каталогами; інакше друкує `Usage: ...` і завершується з кодом 1.
2. Якщо існує `/tmp/backup.lock` — інша копія вже працює: друкується `Backup already running` і скрипт виходить. Lock створюється лише після перевірки, а `trap ... EXIT` гарантує його видалення за будь-якого сценарію завершення (успіх чи помилка), тож «мертвий» lock не залишається.
3. Усі файли з каталогу логів пакуються у `logs_backup_YYYY-MM-DD_HH-MM.tar.gz` в каталозі бекапів (`tar -C` архівує вміст без повних шляхів).
4. За кодом виходу `tar` скрипт друкує або `Backup created: <повний шлях>` (через `realpath`), або `Backup failed` і завершується з кодом 2.

## Перевірка роботи з різними аргументами

### Тест 1. Без аргументів
```bash
./backup.sh
echo "exit code: $?"
```
```text
Usage: ./backup.sh <log_dir> <backup_dir>
exit code: 1
```

### Тест 2. Неіснуючий каталог
```bash
./backup.sh /tmp/no_such_dir ./backup_dir
echo "exit code: $?"
```
```text
Usage: ./backup.sh <log_dir> <backup_dir>
exit code: 1
```

### Тест 3. Успішний бекап
```bash
mkdir -p logs backup_dir
echo "log line 1" > logs/app.log
echo "err line" > logs/error.log
dmesg | head -20 > logs/kernel.log
./backup.sh ./logs ./backup_dir
echo "exit code: $?"
ls -l backup_dir/
```
```text
Backup created: /home/vladyslav/hw6/backup_dir/logs_backup_2026-07-04_06-42.tar.gz
exit code: 0
total 4
-rw-r--r-- 1 vladyslav vladyslav 210 Jul  4 06:42 logs_backup_2026-07-04_06-42.tar.gz
```

Вміст архіву:
```bash
tar -tzf backup_dir/logs_backup_2026-07-04_06-42.tar.gz
```
```text
./
./app.log
./error.log
./kernel.log
```

### Тест 4. Захист від паралельного запуску
```bash
touch /tmp/backup.lock
./backup.sh ./logs ./backup_dir
echo "exit code: $?"
rm /tmp/backup.lock
```
```text
Backup already running
exit code: 0
```

### Тест 5. Помилка архівації (каталог бекапів без права запису)
```bash
mkdir -p backup_ro && chmod 555 backup_ro
./backup.sh ./logs ./backup_ro
echo "exit code: $?"
ls /tmp/backup.lock
```
```text
tar (child): ./backup_ro/logs_backup_2026-07-04_06-43.tar.gz: Cannot open: Permission denied
tar (child): Error is not recoverable: exiting now
tar: Child returned status 2
tar: Error is not recoverable: exiting now
Backup failed
exit code: 2
ls: cannot access '/tmp/backup.lock': No such file or directory
```
Навіть після помилки lock-файл прибрано (`trap ... EXIT` спрацював) — наступний запуск не буде хибно заблоковано.
