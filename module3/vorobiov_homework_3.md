# Воробйов Владислав homework 3

## Завдання 1. Огляд активних процесів

### 1.1 Список усіх процесів через `ps`
```bash
ps aux | head -15
ps aux | wc -l
```

Результат (перші 14 рядків + загальна кількість процесів):
```text
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0  27072 17548 ?        Ss   Apr27   1:34 /sbin/init splash
root           2  0.0  0.0      0     0 ?        S    Apr27   0:04 [kthreadd]
root           3  0.0  0.0      0     0 ?        S    Apr27   0:00 [pool_workqueue_release]
root           4  0.0  0.0      0     0 ?        I<   Apr27   0:00 [kworker/R-rcu_gp]
root           5  0.0  0.0      0     0 ?        I<   Apr27   0:00 [kworker/R-sync_wq]
root           6  0.0  0.0      0     0 ?        I<   Apr27   0:00 [kworker/R-kvfree_rcu_reclaim]
root           7  0.0  0.0      0     0 ?        I<   Apr27   0:00 [kworker/R-slub_flushwq]
root           8  0.0  0.0      0     0 ?        I<   Apr27   0:00 [kworker/0:0H-events_highpri]
root          10  0.0  0.0      0     0 ?        I    Apr27   0:00 [kworker/u160:0-ipv6_addrconf]
root          12  0.0  0.0      0     0 ?        I    Apr27   0:00 [kworker/u160:1-ipv6_addrconf]
root          13  0.0  0.0      0     0 ?        I<   Apr27   0:00 [kworker/R-mm_percpu_wq]
root          14  0.0  0.0      0     0 ?        I    Apr27   0:01 [kworker/R-rcu_preempt]
root          15  0.0  0.0      0     0 ?        S    Apr27   2:29 [ksoftirqd/0]
645
```

### 1.2 Процес із найбільшим використанням пам'яті
Інтерактивно: запустити `top` (або `htop`), натиснути `Shift+M` для сортування за `%MEM`. Тут наводжу неінтерактивний еквівалент:
```bash
top -b -n 1 -o %MEM | head -12
```

Результат:
```text
top - 19:02:23 up 7 days, 19:37,  1 user,  load average: 1.83, 1.73, 1.54
Tasks: 644 total, 2 running, 635 sleep, 1 d-sleep, 0 stopped, 6 zombie
%Cpu(s):  3.9 us,  2.2 sy,  0.0 ni, 93.5 id,  0.0 wa,  0.2 hi,  0.2 si,  0.0 st
MiB Mem :  64311.3 total,  20654.9 free,   6618.5 used,  37869.3 buff/cache
MiB Swap:  68407.9 total,  68003.7 free,    404.2 used.  57692.8 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
3844983 vlad      20   0 9450720   8.0g   8.0g R  93.3  12.8   0:40.00 git
   1377 root      20   0 2436036   1.0g 193616 S   4.4   1.7    36:46.0 k3s-server
   1269 vlad      20   0   18.6g 846876  86856 S   8.9   1.3 114:39.08 openclaw-gateway
3843538 vlad      20   0   71.2g 398080 170008 S  44.5   0.6   0:24.17 claude
   1871 vlad      20   0   49.2g 339244 247800 S   0.0   0.5   9:28.23 chromium
```

Найбільший споживач RAM — процес `git` (PID 3844983, RES 8.0 GiB, 12.8% від загальних 64 GiB).

### 1.3 PID поточної оболонки
```bash
echo "shell PID: $$"
ps -p $$ -o pid,ppid,comm
```

Результат:
```text
shell PID: 3845681
    PID    PPID COMMAND
3845681 3843538 zsh
```

## Завдання 2. Робота у фоні та керування процесами

### 2.1 Запуск довгої команди у фоні і перегляд `jobs`
```bash
sleep 1000 &
jobs -l
ps -p $! -o pid,stat,comm
```

Результат:
```text
[1]+ 3846091 Running                    sleep 1000 &
    PID STAT COMMAND
3846091 S    sleep
```

### 2.2 Повернення процесу на передній план і призупинення
В інтерактивному терміналі: `fg %1` — повертає на передній план, потім `Ctrl+Z` зупиняє (`SIGTSTP`), `bg %1` повертає у фон. У неінтерактивному скрипті еквівалентно `kill -STOP` / `kill -CONT`:
```bash
kill -STOP $PID            # еквівалент Ctrl+Z (SIGSTOP)
jobs -l
ps -p $PID -o pid,stat,comm
kill -CONT $PID            # еквівалент bg (SIGCONT)
ps -p $PID -o pid,stat,comm
```

Результат:
```text
[1]+ 3846091 Stopped (signal)           sleep 1000
    PID STAT COMMAND
3846091 T    sleep
    PID STAT COMMAND
3846091 S    sleep
```
Стан `T` — процес зупинено, `S` — знову у фоні (sleeping).

### 2.3 Завершення процесу через `kill` і `kill -9`
```bash
kill $PID                  # SIGTERM
sleep 0.3
ps -p $PID -o pid,stat,comm 2>&1 || echo "process gone"

sleep 1000 & PID2=$!
kill -9 $PID2              # SIGKILL — примусове завершення
jobs -l
```

Результат:
```text
process gone
[2]+ 3846119 Running                    sleep 1000 &
[2]+ Killed                     sleep 1000
```

### 2.4 Запуск через `nohup`
```bash
cd /tmp
nohup bash -c 'for i in 1 2 3 4 5; do echo "tick $i $(date +%T)"; sleep 1; done' \
    > hw3_nohup.log 2>&1 &
ps -p $! -o pid,ppid,stat,comm
# ... через ~6 секунд:
cat hw3_nohup.log
```

Результат:
```text
    PID    PPID STAT COMMAND
3846222 3846202 SN   bash
tick 1 19:02:58
tick 2 19:02:59
tick 3 19:03:00
tick 4 19:03:01
tick 5 19:03:02
```
`nohup` ігнорує `SIGHUP`, тому процес продовжує працювати після закриття термінала; стандартні потоки перенаправлено у файл (`nohup.out` за замовчуванням, тут — `hw3_nohup.log`).

## Завдання 3. Пріоритети та обмеження

### 3.1 Запуск з підвищеним `nice` (нижчий пріоритет)
```bash
nice -n 10 sleep 300 &
ps -o pid,ni,pri,stat,comm -p $!
```

Результат:
```text
    PID  NI PRI STAT COMMAND
3846437  10   9 SN   sleep
```
Колонка `NI=10` — підвищене значення nice; `STAT=SN` — `N` означає nice > 0.

### 3.2 Зміна пріоритету вже запущеного процесу
```bash
renice -n 15 -p 3846437
ps -o pid,ni,pri,stat,comm -p 3846437
```

Результат:
```text
3846437 (process ID) old priority 10, new priority 15
    PID  NI PRI STAT COMMAND
3846437  15   4 SN   sleep
```
Без root звичайний користувач може лише підвищувати `nice` (робити процес менш пріоритетним); зменшити nice назад до < 0 заборонено (`ulimit -e` у наступному пункті це підтверджує: `max nice = 0`).

### 3.3 Поточні обмеження ресурсів через `ulimit`
```bash
ulimit -a
```

Результат:
```text
-t: cpu time (seconds)              unlimited
-f: file size (blocks)              unlimited
-d: data seg size (kbytes)          unlimited
-s: stack size (kbytes)             8192
-c: core file size (blocks)         unlimited
-m: resident set size (kbytes)      unlimited
-u: processes                       256698
-n: file descriptors                524288
-l: locked-in-memory size (kbytes)  8192
-v: address space (kbytes)          unlimited
-x: file locks                      unlimited
-i: pending signals                 256698
-q: bytes in POSIX msg queues       819200
-e: max nice                        0
-r: max rt priority                 0
-N 15: rt cpu time (microseconds)   unlimited
```

## Завдання 4. Моніторинг ресурсів

### 4.1 Дисковий простір
```bash
df -h
```

Результат:
```text
Filesystem        Size  Used Avail Use% Mounted on
dev                32G     0   32G   0% /dev
run                32G  3.2M   32G   1% /run
efivarfs          384K   94K  286K  25% /sys/firmware/efi/efivars
/dev/mapper/root  231G  202G   28G  88% /
tmpfs              32G   43M   32G   1% /dev/shm
tmpfs              32G  211M   32G   1% /tmp
/dev/sda1         2.0G  301M  1.8G  15% /boot
/dev/mapper/root  231G  202G   28G  88% /home
/dev/mapper/root  231G  202G   28G  88% /var/log
/dev/mapper/root  231G  202G   28G  88% /var/cache/pacman/pkg
tmpfs             6.3G  292K  6.3G   1% /run/user/1000
```
Кореневий розділ заповнений на 88% (202G / 231G).

### 4.2 Оперативна пам'ять у зручному форматі
```bash
free -h
```

Результат:
```text
               total        used        free      shared  buff/cache   available
Mem:            62Gi       6.5Gi        19Gi       182Mi        37Gi        56Gi
Swap:           66Gi       404Mi        66Gi
```
Вільно (`available`) ~56 GiB з 62 GiB, swap майже не використовується (404 MiB з 66 GiB).
