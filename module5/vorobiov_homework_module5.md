# Воробйов Владислав homework module 5

Тестове середовище: два Ubuntu 24.04 хости в одній мережі — «сервер» (192.168.148.2, встановлено `openssh-server`, користувач `student`) і «клієнт» (192.168.148.3), з якого виконуються всі команди:
```bash
docker network create hw5net
docker run -d --name hw5-server --hostname server --network hw5net ubuntu:24.04 sleep infinity
docker run -d --name hw5-client --hostname client --network hw5net ubuntu:24.04 sleep infinity
```

## Завдання 1. Мережева діагностика

### 1.1 IP-адреси та інтерфейси
```bash
ip a
```

Результат:
```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if43: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether fa:87:1f:1f:5c:84 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.148.3/24 brd 192.168.148.255 scope global eth0
       valid_lft forever preferred_lft forever
```

### 1.2 Доступність публічного вузла
```bash
ping -c 4 8.8.8.8
```

Результат:
```text
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=23.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=115 time=19.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=115 time=19.8 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=115 time=19.7 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3014ms
rtt min/avg/max/mdev = 19.393/20.628/23.628/1.738 ms
```

### 1.3 Відкриті listening-порти
На клієнті:
```bash
ss -tulpn
```
```text
Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess
udp   UNCONN 0      0         127.0.0.11:55328      0.0.0.0:*
tcp   LISTEN 0      4096      127.0.0.11:44575      0.0.0.0:*
```

На сервері:
```bash
ss -tulpn
```
```text
Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess
udp   UNCONN 0      0         127.0.0.11:36909      0.0.0.0:*
tcp   LISTEN 0      4096      127.0.0.11:37699      0.0.0.0:*
tcp   LISTEN 0      128          0.0.0.0:22         0.0.0.0:*    users:(("sshd",pid=3718,fd=3))
tcp   LISTEN 0      128             [::]:22            [::]:*    users:(("sshd",pid=3718,fd=4))
```

Підсумок:
- локальна IP-адреса інтерфейсу `eth0` клієнта — **192.168.148.3/24**;
- доступ до інтернету є — 4/4 пакети до 8.8.8.8, 0% втрат, ~20 мс;
- приклад сервісу, який слухає порт — **sshd на порту 22** (на сервері, слухає `0.0.0.0:22`).

## Завдання 2. SSH-доступ з ключами та config

### 2.1 Генерація SSH-ключа
```bash
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519
```

Результат:
```text
Generating public/private ed25519 key pair.
Your identification has been saved in /root/.ssh/id_ed25519
Your public key has been saved in /root/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:vUPer9l6H/rHGT3Pb6Ow/j4sxNBe1bI1Qh9WtRRZfbM root@client
```

### 2.2 Копіювання ключа на сервер
В інтерактивному терміналі `ssh-copy-id` один раз запитує пароль користувача; тут пароль передано неінтерактивно через `sshpass`:
```bash
sshpass -p 'Student2026' ssh-copy-id -o StrictHostKeyChecking=accept-new student@192.168.148.2
```

Результат:
```text
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh -o 'StrictHostKeyChecking=accept-new' 'student@192.168.148.2'"
and check to make sure that only the key(s) you wanted were added.
```

### 2.3 Файл `~/.ssh/config`
```bash
cat ~/.ssh/config
```
```text
Host myserver
    HostName 192.168.148.2
    User student
    IdentityFile ~/.ssh/id_ed25519
```

### 2.4 Підключення короткою командою без пароля
```bash
ssh -o BatchMode=yes myserver "hostname && whoami"
```

Результат:
```text
server
student
```
`BatchMode=yes` взагалі забороняє запит пароля, тож успішне виконання доводить, що автентифікація пройшла тільки за ключем.

Підсумок:
- ім'я Host у config — **myserver**;
- підключення без пароля **працює** (перевірено з `BatchMode=yes`).

## Завдання 3. Копіювання файлів між машинами

### 3.1 Створення тестового файлу та передача через `scp`
```bash
echo "test" > test.txt
scp test.txt myserver:~/
```

### 3.2 Директорія для синхронізації на сервері
```bash
ssh myserver "mkdir -p ~/sync_dir"
```

### 3.3 Синхронізація локальної папки через `rsync`
```bash
mkdir -p sync_local
echo "file one" > sync_local/one.txt
echo "file two" > sync_local/two.txt
rsync -av sync_local/ myserver:~/sync_dir/
```

Результат:
```text
sending incremental file list
./
one.txt
two.txt

sent 202 bytes  received 57 bytes  172.67 bytes/sec
total size is 18  speedup is 0.07
```

### 3.4 Перевірка через `sftp`
```bash
sftp myserver
sftp> ls -l
sftp> ls -l sync_dir
```

Результат:
```text
Connected to myserver.
sftp> ls -l
drwxr-xr-x    ? student  student        28 Jul  4 06:41 sync_dir
-rw-r--r--    ? student  student         5 Jul  4 06:41 test.txt
sftp> ls -l sync_dir
-rw-r--r--    ? student  student         9 Jul  4 06:41 sync_dir/one.txt
-rw-r--r--    ? student  student         9 Jul  4 06:41 sync_dir/two.txt
```

Підсумок:
- файли на сервері: `/home/student/test.txt` (через `scp`) та `/home/student/sync_dir/one.txt`, `/home/student/sync_dir/two.txt` (через `rsync`);
- для перевірки використано `sftp myserver` з командами `ls -l` та `ls -l sync_dir`.
