# ДЗ №1. Изучение средств защиты ОС GNU/Linux

**Уникальный номер:** 372770 \
**ОС:** RED OS release (8.0.3) SERVER \
**Дата:** 01.11.2026

---

## 📦 Артефакты

| Файл | Раздел | Что проверяется |
|---|---|---|
| `mephi-screenshot.png` | 6.1 | Визуальное подтверждение (номер `372770` в терминале) |
| `history.out` | Все | История выполненных команд |
| `log.out` | Все | Полный лог терминальной сессии (доп.) |
| `stat.out` | 1, 3, 4 | `stat` для `/home/user1`, `~/mycat`, `~/mychown` |
| `getcap.out` | 4 | Вывод `getcap ~/mychown` → `cap_chown=ep` |
| `setuid_files.out` | 2.1 | Результат `find / -perm -4000 -type f` (30 SUID-файлов) |
| `privileged_processes.out` | 2.2 | Процессы с EUID=0 и RUID обычного пользователя |
| `passwd` | 1 | Копия `/etc/passwd` — запись `user1:x:1234:1002:...` |
| `shadow` | 1 | Копия `/etc/shadow` — политика пароля `user1` (90 дней) |
| `group` | 1 | Копия `/etc/group` — группа `students` (GID 1002) |
| `sudoers` | 5 | Копия `/etc/sudoers` — правило `user1 ALL=(root) /usr/bin/date` |

---
---

## Ход выполнения

### Раздел 1. Создание пользователя

```bash
sudo groupadd students
sudo useradd -u 1234 -g students -m -s /bin/bash user1
sudo passwd user1
sudo chage -M 90 user1
```

Проверка:

```bash
$ id user1
uid=1234(user1) gid=1002(students) группы=1002(students)

$ sudo chage -l user1
Максимальное количество дней между сменой пароля : 90
```

**Результат:** пользователь создан, UID 1234, группа `students`, смена пароля каждые 90 дней.

---

### Раздел 2. Мониторинг файлов и процессов

**2.1. SUID-файлы:**

```bash
sudo find / -perm -4000 -type f 2>/dev/null > ~/setuid_files.out
```

Найдено **30 SUID-файлов** (в т.ч. `/usr/bin/passwd`, `/usr/bin/sudo`, `/usr/bin/mount`).

**2.2. Процессы с EUID=0 и RUID ≥ 1000:**

```bash
ps -eo pid,ruid,euid,comm --sort=pid | awk '$2 >= 1000 && $3 == 0' > ~/privileged_processes.out
```

Во втором терминале запущен `passwd` (остановлен на вводе нового пароля). Результат:

```
110995  1001  0  passwd
```

---

### Раздел 3. Механизм set-UID

```bash
cp /usr/bin/cat ~/mycat
sudo chown root:root ~/mycat
sudo chmod u+s ~/mycat

$ ls -l ~/mycat
-rwsr-xr-x. 1 root root 36568 ... /home/vladblednov/mycat

$ ~/mycat /etc/shadow | head -n 1
root:$y$j9T$VjqzT1iLAZkFyDS.fk/OuO/.$4jIH..SP4RfBAhZdQuygtQEAUkJ00TbJFgA/slmeFQB::0:99999:7:::
```

**Результат:** SUID установлен, обычный пользователь прочитал `/etc/shadow`.

---

### Раздел 4. Механизм привилегий

```bash
cp /usr/bin/chown ~/mychown
sudo chown root:root ~/mychown
sudo setcap cap_chown=ep ~/mychown

$ getcap ~/mychown
/home/vladblednov/mychown cap_chown=ep

$ touch ~/testfile
$ ~/mychown user1 ~/testfile
$ ls -l ~/testfile
-rw-r--r--. 1 user1 vladblednov 0 ... /home/vladblednov/testfile
```

**Результат:** capability `cap_chown=ep` установлена, владелец файла сменён на `user1`.

Повторный `touch ~/testfile` вернул `Отказано в доступе` - так как файл уже принадлежит `user1`, обычный пользователь не может его изменить. Это подтверждает, что `cap_chown` сработала.

---

### Раздел 5. Механизм sudo

```bash
sudo SUDO_EDITOR=nano visudo
# В конец /etc/sudoers добавлена строка:
#   user1 ALL=(root) /usr/bin/date

$ sudo -l -U user1
Пользователь user1 может выполнять следующие команды на red-lab:
    (root) /usr/bin/date
```

**Результат:** правило применено, `user1` может выполнять `/usr/bin/date` от root.

---

### Раздел 6. Сбор артефактов

```bash
stat /home/user1 > ~/stat.out
stat ~/mycat   >> ~/stat.out
stat ~/mychown >> ~/stat.out
getcap ~/mychown > ~/getcap.out
history > ~/history.out
```

Скриншот с номером `372770` сохранён как `mephi-screenshot.png`.

Копии системных файлов:

```bash
sudo cp /etc/passwd  ./passwd
sudo cp /etc/shadow  ./shadow
sudo cp /etc/group   ./group
sudo cp /etc/sudoers ./sudoers
sudo chown $USER:$USER ./passwd ./shadow ./group ./sudoers
```

---

## Примечания

- Вход в систему - по SSH-ключу (без пароля). `sudo` запрашивает пароль пользователя.
- Все изменения сохраняются после перезагрузки (системные файлы `/etc/*`, SUID на `~/mycat`, capability на `~/mychown`, правило в `/etc/sudoers`).

---

## Соответствие заданию

| Раздел | Требование | Выполнено |
|---|---|---|
| 1 | `user1`, UID 1234, группа `students`, пароль 90 дней | ✅ |
| 2.1 | Все SUID-файлы | ✅ (30) |
| 2.2 | Процессы с EUID=0, RUID ≥ 1000 | ✅ (`passwd`) |
| 3 | SUID + привилегированная операция | ✅ (`cat` → `/etc/shadow`) |
| 4 | Capability + привилегированная операция | ✅ (`chown` → смена владельца) |
| 5 | `sudo` для `user1` на изменение времени | ✅ (`/usr/bin/date`) |
| 6 | Все артефакты | ✅ |

