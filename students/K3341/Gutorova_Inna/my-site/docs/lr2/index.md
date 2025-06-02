
# Лабораторная работа 2: Потоки, процессы и асинхронность в Python


## Задача 1: Вычисление суммы чисел

### 1. Threading (потоки)

```
def threading_sum():
    num_threads = 4
    result = [0] * num_threads  # Общий mutable объект для результатов
    threads = []

    for i in range(num_threads):
        # Создание и запуск потоков
        thread = threading.Thread(target=calculate_sum, 
                               args=(start, end, result, i))
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()  # Ожидание завершения всех потоков

    total_sum = sum(result)
```

**Особенности:**
- Используется общий список `result` для сбора результатов
- Явное создание и запуск каждого потока
- Обязательное использование `join()` для синхронизации

### 2. Multiprocessing (процессы)

```
def multiprocessing_sum():
    with multiprocessing.Pool(processes=num_processes) as pool:
        # Распределение работы между процессами
        results = pool.map(calculate_sum, ranges)
    
    total_sum = sum(results)
```

**Особенности:**
- Используется `Pool` для управления пулом процессов
- `map` автоматически распределяет задачи
- Нет необходимости в ручном управлении процессами
- Процессы полностью изолированы (нет shared memory)

### 3. Async (асинхронность)

```
async def async_sum():
    tasks = []
    for i in range(num_tasks):
        # Создание асинхронных задач
        task = asyncio.create_task(calculate_sum(start, end))
        tasks.append(task)

    # Параллельное выполнение задач
    results = await asyncio.gather(*tasks)
    total_sum = sum(results)
```

**Особенности:**
- Используется `create_task` для планирования корутин
- `gather` ожидает завершения всех задач
- Все выполняется в одном потоке
- Нет реального параллелизма (кооперативная многозадачность)

---

## Задача 2: Парсинг веб-страниц

### Источники данных
Для парсинга были выбраны issue-трекеры популярных GitHub-репозиториев:
1. [VS Code](https://github.com/microsoft/vscode/issues)
2. [React](https://github.com/facebook/react/issues)
3. [TensorFlow](https://github.com/tensorflow/tensorflow/issues)

### Парсинг данных
Для каждого репозитория извлекалась следующая информация:

```
{
    "title": "Название issue",
    "number": "Номер issue (#12345)",
    "status": "open/closed",
    "author": "Автор issue",
    "created_date": "Дата создания",
    "labels": ["bug", "enhancement", ...]
}
```

#### Пример реального спарсенного issue:
```
{
    "title": "Improve error handling in component X",
    "number": "#21543",
    "status": "open",
    "author": "developer123",
    "created_date": "2023-05-15",
    "labels": ["bug", "good first issue"]
}
```

### Алгоритм парсинга
1. **Получение HTML**:
```
   # Для синхронных версий
   response = requests.get(issues_url, headers=headers)
   soup = BeautifulSoup(response.text, 'html.parser')
   
   # Для async версии
   async with session.get(issues_url) as response:
       html = await response.text()
       soup = BeautifulSoup(html, 'html.parser')
```

2. **Извлечение данных**:
   - Поиск элементов по CSS-селекторам:
   
```
   item.select('.ListItems-module__listItem--Blv7W')  # Контейнеры issues
   item.select_one('.IssuePullRequestTitle-module__title')  # Заголовок
```

3. **Обработка даты**:
   Специальная функция для конвертации строковых дат GitHub:
```
   def parse_date(date_str):
       # Пример входных данных: "on May 15, 2023"
       clean_str = date_str.replace("on ", "")
       return datetime.strptime(clean_str, "%b %d, %Y")
```

Процесс сохранения:
1. Создание задачи:
```
   task = Task(
       title=f"{repo_name} #{issue_number}",
       description=f"Issue: {title}\nStatus: {status}...",
       priority=Priority.high if 'bug' in labels else Priority.medium,
       status=Status.todo
   )
   session.add(task)
```

2. Обработка тегов (многие-ко-многим):
```
   for label in labels:
       tag = session.query(Tag).filter_by(name=label).first()
       if not tag:
           tag = Tag(name=label)
           session.add(tag)
       session.add(TaskTag(task_id=task.id, tag_id=tag.id))
```

### Особенности реализации для каждого подхода

1. **Threading**:
   - Общая сессия БД между потоками
   - Риск race condition при работе с БД
   - Требуется явная синхронизация (не реализована)

2. **Multiprocessing**:
   - Каждый процесс имеет свою сессию БД
   - Автоматическая изоляция данных
   - Высокие накладные расходы на создание процессов

3. **Async**:
   - Асинхронная сессия БД (AsyncSession)
   - Использование `await` для всех IO-операций
   - Наиболее эффективное решение для данной задачи


### 1. Threading

```
def main():
    threads = []
    for repo in REPOSITORIES:
        # Создание потока для каждого URL
        thread = threading.Thread(target=parse_and_save, args=(repo,))
        thread.start()
        threads.append(thread)

    for thread in threads:
        thread.join()  # Ожидание завершения
```

**Проблемы:**
- При большой нагрузке может создать слишком много потоков
- Блокирующие IO-операции
- Потенциальные race condition при работе с БД

### 2. Multiprocessing

```
def main():
    processes = []
    for repo in REPOSITORIES:
        # Создание отдельного процесса
        process = multiprocessing.Process(
            target=parse_and_save,
            args=(repo,)
        )
        process.start()
        processes.append(process)

    for process in processes:
        process.join()  # Синхронизация
```

**Особенности:**
- Полная изоляция процессов
- Высокие накладные расходы на создание
- Требуется отдельное соединение с БД для каждого процесса

### 3. Async

```
async def main():
    async with aiohttp.ClientSession() as session:
        # Создание задач для всех URL
        tasks = [parse_and_save(repo, session) for repo in REPOSITORIES]
        await asyncio.gather(*tasks)  # Параллельное выполнение
```

**Преимущества:**
- Одна сессия для всех запросов
- Нет накладных расходов на создание потоков/процессов
- Наиболее эффективное использование ресурсов для I/O операций

## Ключевые различия в организации `main()`

| Аспект              | Threading               | Multiprocessing         | Async                 |
|---------------------|-------------------------|-------------------------|-----------------------|
| Создание задач      | `Thread(target=...)`    | `Process(target=...)`   | `create_task()`       |
| Запуск              | `.start()`              | `.start()`              | Не требуется (авто)   |
| Ожидание завершения | `.join()`               | `.join()`               | `await gather()`      |
| Параллелизм         | Псевдо- (GIL)           | Настоящий               | Кооперативный         |
| Обмен данными       | Общая память            | IPC (межпроцессное)     | Через event loop      |
| Подходит для        | I/O с блокировками      | CPU-intensive           | I/O без блокировок    |

## Выводы

1. **Threading**:
   - Простая модель для I/O-bound задач
   - Требует ручного управления потоками
   - Проблемы с GIL и синхронизацией

2. **Multiprocessing**:
   - Настоящий параллелизм для CPU-bound задач
   - Высокие накладные расходы
   - Сложности с разделяемыми ресурсами

3. **Async**:
   - Максимальная эффективность для I/O
   - Требует переписывания кода под async/await
   - Нет реального параллелизма вычислений

