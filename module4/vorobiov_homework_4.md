# Воробйов Владислав homework 4

## Завдання 1. Менеджери пакетів

### 1.1 Оновлення списку пакетів
```bash
sudo apt update
```

Результат (останні рядки):
```text
Hit:2 http://ports.ubuntu.com/ubuntu-ports resolute-updates InRelease
Hit:3 http://ports.ubuntu.com/ubuntu-ports resolute-security InRelease
Reading package lists...
Building dependency tree...
Reading state information...
7 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

### 1.2 Встановлення утиліти `tree`
```bash
sudo apt install -y tree
```

Результат:
```text
Get:1 http://ports.ubuntu.com/ubuntu-ports resolute/universe arm64 tree arm64 2.3.1-1 [52.7 kB]
Fetched 52.7 kB in 0s (600 kB/s)
Selecting previously unselected package tree.
Preparing to unpack …/tree_2.3.1-1_arm64.deb…
Unpacking tree (2.3.1-1)…
Setting up tree (2.3.1-1)…
```

### 1.3 Перевірка встановлення та версії
```bash
tree --version
apt policy tree
```

Результат:
```text
tree v2.3.1 © 1996 - 2026 by Steve Baker, Thomas Moore, Francesc Rocher, Florian Sesser, Kyosuke Tokoro
tree:
  Installed: 2.3.1-1
  Candidate: 2.3.1-1
  Version table:
 *** 2.3.1-1 500
        500 http://ports.ubuntu.com/ubuntu-ports resolute/universe arm64 Packages
        100 /var/lib/dpkg/status
```
Пакет встановлено, версія 2.3.1-1 (`Installed: 2.3.1-1`).

### 1.4 Видалення пакета
```bash
sudo apt remove -y tree
tree --version
apt policy tree
```

Результат:
```text
Summary:
  Upgrading: 0, Installing: 0, Removing: 1, Not Upgrading: 7
  Freed space: 162 kB
Removing tree (2.3.1-1)…

bash: /usr/bin/tree: No such file or directory
tree:
  Installed: (none)
  Candidate: 2.3.1-1
```
Після видалення `Installed: (none)` — пакета в системі більше немає.

## Завдання 2. Керування сервісами через systemctl

### 2.1 Статус сервісу `ssh`
```bash
systemctl status ssh
```

Результат:
```text
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)
     Active: active (running) since Sat 2026-07-04 06:37:32 UTC; 27s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 3002 (sshd)
      Tasks: 1 (limit: 10833)
     Memory: 3M (peak: 3.9M)
        CPU: 16ms
```

### 2.2 Зупинка сервісу та перевірка, що він не активний
```bash
sudo systemctl stop ssh
systemctl is-active ssh
```

Результат:
```text
inactive
```

### 2.3 Повторний запуск сервісу
```bash
sudo systemctl start ssh
systemctl is-active ssh
systemctl status ssh | head -5
```

Результат:
```text
active
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)
     Active: active (running) since Sat 2026-07-04 06:37:59 UTC; 7ms ago
```

### 2.4 Додавання сервісу в автозавантаження
```bash
sudo systemctl enable ssh
systemctl is-enabled ssh
```

Результат:
```text
Synchronizing state of ssh.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable ssh
Created symlink '/etc/systemd/system/sshd.service' → '/usr/lib/systemd/system/ssh.service'.
Created symlink '/etc/systemd/system/multi-user.target.wants/ssh.service' → '/usr/lib/systemd/system/ssh.service'.
enabled
```
Створено symlink у `multi-user.target.wants` — тепер сервіс стартує разом із системою.

## Завдання 3. Робота з логами

### 3.1 Останні 10 рядків `/var/log/syslog`
```bash
cd /var/log
sudo tail -10 syslog
```

Результат:
```text
2026-07-04T06:37:59.761123+00:00 os-hw systemd[1]: Started ssh.service - OpenBSD Secure Shell server.
2026-07-04T06:37:59.781578+00:00 os-hw systemd[1]: Reload requested from client PID 3481 ('systemctl')...
2026-07-04T06:37:59.781737+00:00 os-hw systemd[1]: Reloading...
2026-07-04T06:37:59.933530+00:00 os-hw systemd[1]: Reloading finished in 151 ms.
2026-07-04T06:37:59.945067+00:00 os-hw systemd[1]: Reload requested from client PID 3591 ('systemctl')...
2026-07-04T06:37:59.945141+00:00 os-hw systemd[1]: Reloading...
2026-07-04T06:38:00.079671+00:00 os-hw systemd[1]: Reloading finished in 135 ms.
2026-07-04T06:38:00.088174+00:00 os-hw systemd[1]: Reload requested from client PID 3477 ('systemctl')...
2026-07-04T06:38:00.088237+00:00 os-hw systemd[1]: Reloading...
2026-07-04T06:38:00.280960+00:00 os-hw systemd[1]: Reloading finished in 192 ms.
```

### 3.2 Тільки помилки через `journalctl`
```bash
journalctl -p err -b
```

Результат:
```text
Jul 04 06:36:20 os-hw kernel: virtio_snd virtio3: control message (0x00000100) timeout
Jul 04 06:36:20 os-hw kernel: virtio_snd virtio3: probe with driver virtio_snd failed with error -110
```
З рівнем `err` у поточному завантаженні лише дві помилки ядра (драйвер віртуального звукового пристрою, на роботу системи не впливає).

### 3.3 Записи про запуск/зупинку сервісу із Завдання 2
```bash
journalctl -u ssh | grep -E "Started|Stopped"
```

Результат:
```text
Jul 04 06:37:32 os-hw systemd[1]: Started ssh.service - OpenBSD Secure Shell server.
Jul 04 06:37:59 os-hw systemd[1]: Stopped ssh.service - OpenBSD Secure Shell server.
Jul 04 06:37:59 os-hw systemd[1]: Started ssh.service - OpenBSD Secure Shell server.
```
Видно повний цикл із Завдання 2: перший запуск, зупинка (`systemctl stop`) і повторний запуск (`systemctl start`).

## Завдання 4. Створення власного сервісу

### 4.1 Скрипт, що щосекунди записує дату у файл
```bash
cat > ~/date-logger.sh << "EOF"
#!/bin/bash
while true; do
    date >> /home/vladyslav/date-log.txt
    sleep 1
done
EOF
chmod +x ~/date-logger.sh
```

### 4.2 Файл конфігурації сервісу
```bash
sudo tee /etc/systemd/system/myscript.service << "EOF"
[Unit]
Description=My date logger script

[Service]
User=vladyslav
ExecStart=/home/vladyslav/date-logger.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

### 4.3 Запуск сервісу і перевірка
```bash
sudo systemctl daemon-reload
sudo systemctl start myscript
systemctl status myscript
```

Результат:
```text
● myscript.service - My date logger script
     Loaded: loaded (/etc/systemd/system/myscript.service; disabled; preset: enabled)
     Active: active (running) since Sat 2026-07-04 06:38:33 UTC; 14ms ago
   Main PID: 3953 (date-logger.sh)
      Tasks: 2 (limit: 10833)
     Memory: 4.6M (peak: 4.6M)
        CPU: 13ms
     CGroup: /system.slice/myscript.service
```

Перевірка, що дані записуються у файл:
```bash
tail -5 ~/date-log.txt
wc -l ~/date-log.txt
sleep 2
wc -l ~/date-log.txt
```

Результат:
```text
Sat Jul  4 06:38:33 AM UTC 2026
Sat Jul  4 06:38:34 AM UTC 2026
Sat Jul  4 06:38:35 AM UTC 2026
Sat Jul  4 06:38:36 AM UTC 2026
Sat Jul  4 06:38:37 AM UTC 2026
5 /home/vladyslav/date-log.txt
7 /home/vladyslav/date-log.txt
```
Сервіс активний, `Main PID` — наш скрипт. Кількість рядків у файлі зросла з 5 до 7 за 2 секунди — дата дописується щосекунди.
