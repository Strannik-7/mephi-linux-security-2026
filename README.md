# Изучение средств защиты ОС GNU/Linux

**Уникальный номер:** 372770
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

