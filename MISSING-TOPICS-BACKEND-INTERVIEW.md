# Анализ вопросов из backend-interview-questions

## Источник
https://github.com/starandtina/backend-interview-questions

## Категории вопросов

### 1. General/Ice Breaker
❌ **Не добавляем** - это не технические вопросы

### 2. Management Questions
❓ **Возможно добавить** - вопросы про менеджмент проектов

#### Вопросы:
- How to tackle a story/task which is difficult to estimate and with high risk?
- How to react to unexpected/frequent requirement changes, considering PoC phase and production phase?

**Статус**: ⚠️ Частично покрыто в:
- `software-development/34-SOFTWARE-ESTIMATION-DEEP-DIVE.md`
- `software-development/36-SOFTWARE-PROJECT-MANAGEMENT-DEEP-DIVE.md`
- `software-development/35-SOFTWARE-REQUIREMENTS-ENGINEERING-DEEP-DIVE.md`

**Рекомендация**: Можно добавить отдельную тему "Project Risk Management" и "Handling Requirement Changes"

---

### 3. Algos, Data Structures, & Computer Science Fundamentals Questions

#### ✅ Уже покрыто:
- Merge sort → `operating-system/08-SORTING-ALGORITHMS-DEEP-DIVE.md`
- Process vs Thread → `operating-system/03-PROCESSES-THREADS-DEEP-DIVE.md`
- Race condition → `operating-system/02-CONCURRENCY-DEEP-DIVE.md`
- Critical section → `operating-system/02-CONCURRENCY-DEEP-DIVE.md`
- IPC → `operating-system/16-INTER-PROCESS-COMMUNICATION-DEEP-DIVE.md`
- Virtual Memory → `operating-system/24-OPERATING-SYSTEM-VIRTUAL-MEMORY-DEEP-DIVE.md`
- Algorithms & Data Structures → `programming/12-ALGORITHMS-DATA-STRUCTURES-DEEP-DIVE.md`
- Linked Lists → `go/25-GO-LINKED-LISTS-DEEP-DIVE.md`
- Stacks & Queues → `go/29-GO-STACK-QUEUE-DEEP-DIVE.md`
- Trees → `go/26-GO-TREES-DEEP-DIVE.md`
- Graphs → `go/27-GO-GRAPHS-ALGORITHMS-DEEP-DIVE.md`
- Dynamic Programming → `go/28-GO-DYNAMIC-PROGRAMMING-DEEP-DIVE.md`

#### ❌ Отсутствует (конкретные алгоритмические задачи):
1. **Rectangle overlap detection** - алгоритм определения пересечения двух прямоугольников и вычисления площади пересечения
2. **Integer partition** - разбиение числа на сумму целых чисел (все возможные последовательности)
3. **Bracket matching** - проверка правильности скобок (O(n) space, лучше O(1))
4. **Reverse words in sentence** - переворот слов в предложении
5. **Nearest greater element** - найти ближайший больший элемент слева (O(n))
6. **Rotated sorted array search** - поиск в повернутом отсортированном массиве
7. **Linked list loop length** - найти длину цикла в связанном списке
8. **Valid parentheses combinations** - все валидные комбинации n пар скобок
9. **Longest common subsequence** - найти самую длинную общую подпоследовательность
10. **Topological sort** - топологическая сортировка
11. **Search in sorted matrix** - поиск в матрице, где каждая строка и столбец отсортированы
12. **rand5 to rand7** - реализовать rand7 используя rand5

#### ❌ Отсутствует (C/C++ специфичные вопросы):
13. **#define macros** - использование #define для констант и функций
14. **do{while(0)} pattern** - зачем оборачивать макросы в do{while(0)}
15. **static, const, volatile, typeof** - объяснение ключевых слов
16. **trim_string function** - реализация функции удаления пробелов
17. **Non-inheritable class (C++)** - класс, который нельзя наследовать
18. **LENGTH_OF macro** - макрос для получения длины массива

**Рекомендация**: Эти вопросы специфичны для C/C++ и алгоритмических задач. Можно добавить:
- "Common Algorithm Problems Deep Dive" - сборник популярных алгоритмических задач
- "C/C++ Programming Fundamentals" - если нужно покрыть C/C++ вопросы

---

### 4. Design Questions

#### ✅ Уже покрыто (общие концепции):
- Queue using stacks → концепция покрыта в `go/29-GO-STACK-QUEUE-DEEP-DIVE.md`
- Job queue → концепция покрыта в `networking/24-MESSAGE-QUEUES-DEEP-DIVE.md`
- Fair queue → концепция покрыта в `networking/24-MESSAGE-QUEUES-DEEP-DIVE.md`
- Atomic updates → концепция покрыта в `database/02-DATABASE-TRANSACTIONS-DEEP-DIVE.md`
- Race conditions → `operating-system/02-CONCURRENCY-DEEP-DIVE.md`
- System design → `software-development/46-SYSTEM-DESIGN-FUNDAMENTALS-DEEP-DIVE.md`

#### ❌ Отсутствует (конкретные дизайн задачи):
1. **In-memory job queue with fairness** - детальный дизайн очереди с ограничениями:
   - Множественные читатели и писатели
   - Ограничение на количество задач одного пользователя
   - Сохранение порядка задач одного пользователя
   - Обработка ошибок

2. **BackupTask design** - дизайн системы бэкапов:
   - Разные стратегии бэкапов (full, differential, log)
   - Загрузка на разные устройства (CIFS, S3, vBlob)
   - Уведомления о результатах
   - Планирование задач
   - Обработка ошибок

3. **Atomic update problem** - детальный анализ проблемы:
   - Сценарии неконсистентности
   - Решения (оптимистичная блокировка, пессимистичная блокировка, версионирование)

4. **Rectangle overlap algorithm** - алгоритм определения пересечения прямоугольников

**Рекомендация**: Можно добавить:
- "Job Queue Design Patterns Deep Dive" - детальный разбор дизайна очередей
- "Backup System Design Deep Dive" - дизайн систем бэкапов
- "Concurrency Control Patterns Deep Dive" - паттерны контроля конкурентности

---

### 5. Backend Linux Questions

#### ✅ Уже покрыто (общие концепции):
- Process management → `operating-system/27-PROCESS-MANAGEMENT-DEEP-DIVE.md`
- File systems → `operating-system/21-FILE-SYSTEMS-DEEP-DIVE.md`
- TCP connections → `networking/01-TCP-IP-DEEP-DIVE.md`
- Git workflow → `software-development/10-GIT-BRANCHING-STRATEGIES-DEEP-DIVE.md`

#### ❌ Отсутствует (конкретные Linux вопросы):
1. **load_avg/buffer/cache metrics** - объяснение метрик Linux
2. **CLOSE_WAIT/TIME_WAIT** - объяснение состояний TCP соединения
3. **Debugging hanging process** - как отлаживать зависший процесс
4. **Debugging iptables rules** - как отлаживать правила iptables
5. **Finding remote port** - как проверить, открыт ли удаленный порт
6. **Finding file descriptors** - как найти количество открытых файловых дескрипторов процесса
7. **Last boot log** - как найти лог последней загрузки
8. **Git commit squashing** - как объединить несколько коммитов
9. **Git hotfix workflow** - workflow для hotfix
10. **umask** - что такое umask
11. **File copying methods** - способы копирования файлов на удаленный сервер
12. **Linux commands** - объяснение команд: ps/sort/awk/join/pwd/iostat/vmstat/top/kill
13. **Network routing through intermediate host** - маршрутизация через промежуточный хост
14. **Batch filename renaming** - массовое переименование файлов в shell
15. **Word frequency analysis** - анализ частоты слов в файле

**Рекомендация**: Можно добавить:
- "Linux System Administration Deep Dive" - администрирование Linux
- "Linux Debugging Techniques Deep Dive" - техники отладки в Linux
- "Linux Network Troubleshooting Deep Dive" - решение сетевых проблем в Linux
- "Shell Scripting Deep Dive" - скрипты shell

---

## Итоговый список недостающих тем

### Высокий приоритет (важные для backend):
1. **Linux System Metrics** (load_avg, buffer, cache)
2. **TCP Connection States** (CLOSE_WAIT, TIME_WAIT)
3. **Linux Debugging Techniques** (hanging processes, iptables)
4. **Job Queue Design Patterns** (fair queue, user limits)
5. **Atomic Update Patterns** (optimistic locking, versioning)

### Средний приоритет:
6. **Backup System Design**
7. **Linux Network Troubleshooting**
8. **Shell Scripting Patterns**
9. **Project Risk Management**
10. **Handling Requirement Changes**

### Низкий приоритет (специфичные алгоритмы):
11. **Common Algorithm Problems** (rectangle overlap, bracket matching, etc.)
12. **C/C++ Programming Fundamentals** (если нужно)

---

## Рекомендации

### Добавить обязательно:
1. Linux System Metrics Deep Dive
2. TCP Connection States Deep Dive (CLOSE_WAIT, TIME_WAIT)
3. Linux Debugging Techniques Deep Dive
4. Job Queue Design Patterns Deep Dive
5. Atomic Update Patterns Deep Dive

### Добавить по желанию:
6. Backup System Design Deep Dive
7. Linux Network Troubleshooting Deep Dive
8. Shell Scripting Deep Dive
9. Project Risk Management Deep Dive
10. Handling Requirement Changes Deep Dive

### Можно пропустить:
- Специфичные алгоритмические задачи (если не фокус на алгоритмы)
- C/C++ вопросы (если не нужен C/C++)

