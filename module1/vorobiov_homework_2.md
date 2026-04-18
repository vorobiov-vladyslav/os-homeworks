# Воробйов Владислав homework 2

## Завдання 1. Ієрархія каталогів Linux

### 1.1 Перехід у кореневий каталог `/` і перегляд вмісту
```bash
cd /
pwd
ls -la
```

Результат:
```text
/
total 24
drwxr-xr-x   1 root root  150 Apr 17 10:11 .
drwxr-xr-x   1 root root  150 Apr 17 10:11 ..
lrwxrwxrwx   1 root root    7 Oct 12  2025 bin -> usr/bin
drwxr-xr-x   6 root root 4096 Jan  1  1970 boot
drwxr-xr-x  20 root root 4040 Apr 18 16:49 dev
drwxr-xr-x   1 root root 3132 Apr 18 16:49 etc
drwxr-xr-x   1 root root   18 Apr 17 12:41 home
lrwxrwxrwx   1 root root    7 Oct 12  2025 lib -> usr/lib
lrwxrwxrwx   1 root root    7 Oct 12  2025 lib64 -> usr/lib
drwxr-xr-x   1 root root    0 Oct 12  2025 mnt
drwxr-xr-x   1 root root   90 Apr 18 10:13 opt
dr-xr-xr-x 622 root root    0 Apr 18 16:49 proc
drwxr-x---   1 root root   42 Apr 18 01:10 root
drwxr-xr-x  26 root root  600 Apr 18 16:49 run
lrwxrwxrwx   1 root root    7 Oct 12  2025 sbin -> usr/bin
drwxr-x---   1 root root  122 Apr 18 16:00 .snapshots
drwxr-xr-x   1 root root   14 Apr 17 09:58 srv
drwxr-xr-x   1 root root   16 Apr 17 10:03 swap
dr-xr-xr-x  13 root root    0 Apr 18 16:49 sys
drwxrwxrwt  17 root root  380 Apr 18 16:51 tmp
drwxr-xr-x   1 root root   94 Apr 17 22:40 usr
drwxr-xr-x   1 root root  116 Apr 18 16:49 var
```

### 1.2 Перехід у `/etc` і перегляд вмісту
```bash
cd /etc
pwd
ls -la
```

Результат:
```text
/etc
total 904
drwxr-xr-x 1 root root   3132 Apr 18 16:49 .
drwxr-xr-x 1 root root    150 Apr 17 10:11 ..
drwxr-xr-x 1 root root     12 Apr 17 09:59 alsa
-rw-r--r-- 1 root root      1 Oct 12  2025 arch-release
-rw-r--r-- 1 root root      0 Apr  9 12:36 arptables.conf
drwxr-xr-x 1 root root      0 Apr 17 09:59 audisp
drwxr-xr-x 1 root root     86 Apr 17 09:59 audit
drwxr-xr-x 1 root root    142 Apr 17 09:59 avahi
-rw-r--r-- 1 root root     28 Dec 11 00:02 bash.bash_logout
-rw-r--r-- 1 root root    733 Dec 11 00:02 bash.bashrc
drwxr-xr-x 1 root root     62 Apr 17 10:01 bash_completion.d
-rw-r--r-- 1 root root    535 Aug 25  2025 bindresvport.blacklist
drwxr-xr-x 1 root root      0 Mar 23 15:30 binfmt.d
dr-xr-xr-x 1 root root     62 Apr 17 07:23 bluetooth
drwxr-xr-x 1 root root     10 Apr 17 07:24 boot
drwxr-xr-x 1 root root     16 Apr 17 10:02 brave
drwxr-xr-x 1 root root     54 Apr 17 09:58 ca-certificates
drwxr-xr-x 1 root root     16 Apr 17 10:02 chromium
drwxr-xr-x 1 root root     24 Apr 17 10:02 cifs-utils
drwxr-xr-x 1 root root     10 Apr 18 10:14 cni
drwxr-xr-x 1 root root     58 Apr 17 10:03 conf.d
-rw-r--r-- 1 root root      0 Apr  9 13:12 conntrackd.conf
drwx------ 1 root root      0 Mar 23 15:30 credstore
...
```

### 1.3 Перехід у `/home` і перегляд списку користувачів
```bash
cd /home
pwd
ls -la
```

Результат:
```text
/home
total 0
drwxr-xr-x  1 root  root   18 Apr 17 12:41 .
drwxr-xr-x   1 root root  150 Apr 17 10:11 ..
drwx------  1 henry henry 114 Apr 18 10:14 henry
drwx--x---+ 1 vlad  vlad  638 Apr 18 16:55 vlad
```

## Завдання 2. Файли, каталоги та посилання

### 2.1 Створення каталогу, файлу, копії, жорсткого та символічного посилань у домашньому каталозі
```bash
mkdir -p /home/vlad/lab2
printf 'Hello from homework 2\n' > /home/vlad/lab2/file.txt
cp /home/vlad/lab2/file.txt /home/vlad/lab2/file-copy.txt
mv /home/vlad/lab2/file-copy.txt /home/vlad/lab2/file-renamed.txt
ln /home/vlad/lab2/file.txt /home/vlad/lab2/file-hardlink.txt
ln -s /home/vlad/lab2/file.txt /home/vlad/lab2/file-symlink.txt
find /home/vlad -maxdepth 2 -name 'file.txt'
ls -lai /home/vlad/lab2
```

Результат:
```text
/home/vlad/lab2/file.txt
total 16
278979 drwxr-xr-x  1 vlad vlad 114 Apr 18 17:17 .
   257 drwx--x---+ 1 vlad vlad 646 Apr 18 17:17 ..
278981 -rw-r--r--  2 vlad vlad  22 Apr 18 17:17 file-hardlink.txt
278982 -rw-r--r--  1 vlad vlad  22 Apr 18 17:17 file-renamed.txt
278983 lrwxrwxrwx  1 vlad vlad  24 Apr 18 17:17 file-symlink.txt -> /home/vlad/lab2/file.txt
278981 -rw-r--r--  2 vlad vlad  22 Apr 18 17:17 file.txt
```

## Завдання 3. Права доступу

### 3.1 Перегляд прав файлу
```bash
ls -l /home/vlad/lab2/file.txt
```

Результат:
```text
-rw-r--r-- 2 vlad vlad 22 Apr 18 17:17 /home/vlad/lab2/file.txt
```

### 3.2 Надання файлу прав тільки на читання
```bash
chmod 444 /home/vlad/lab2/file.txt
ls -l /home/vlad/lab2/file.txt
```

Результат:
```text
-r--r--r-- 2 vlad vlad 22 Apr 18 17:17 /home/vlad/lab2/file.txt
```

### 3.3 Надання власнику права на запис
```bash
chmod u+w /home/vlad/lab2/file.txt
ls -l /home/vlad/lab2/file.txt
```

Результат:
```text
-rw-r--r-- 2 vlad vlad 22 Apr 18 17:17 /home/vlad/lab2/file.txt
```

### 3.4 Перегляд і встановлення `umask`
```bash
umask
umask 022
umask
```

Результат:
```text
022
022
```

## Завдання 4. Користувачі

### 4.1 Створення нового користувача, додавання до sudo-групи та перевірка
```bash
sudo useradd -m -s /bin/bash trainee
sudo usermod -aG wheel trainee
getent passwd trainee
id trainee
getent group wheel
```

Результат:
```text
trainee:x:1002:1004::/home/trainee:/bin/bash
uid=1002(trainee) gid=1004(trainee) groups=1004(trainee),998(wheel)
wheel:x:998:vlad,henry,trainee
```

