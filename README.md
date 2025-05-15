Вот обновленная документация, которая точно соответствует текущей реализации кода:

# Документация gosqllite3 (v0.2)

**Упрощенная библиотека для работы с SQLite3**

## Основные функции

### `connect(file: str)`
Открывает соединение с базой данных SQLite.
- `file` (str) - путь к файлу БД (например, "database.db"). Если файла нет, он будет создан.

### `create(parameters: list, auto_commit: bool = False)`
Создает таблицу с указанными параметрами.
- `parameters` (list) - список, где первый элемент - название таблицы, а второй - строка с определением столбцов (например, `["users", "id INTEGER PRIMARY KEY, name TEXT"]`).
- `auto_commit` (bool) - если True, изменения сохраняются сразу (по умолчанию False).

### `insert(parameters: list, values: list, auto_commit: bool = False)`
Добавляет новую запись в таблицу.
- `parameters` (list) - список из 2 элементов: название таблицы и строка столбцов (можно использовать ~ как плейсхолдер, например `["users", "name~,age~"]`).
- `values` (list) - значения для вставки (соответствуют плейсхолдерам ~).
- `auto_commit` (bool) - автосохранение изменений.

### `selectall(parameters: list, where: bool = False, values: list = None)`
Выбирает данные из таблицы.
- `parameters` (list) - список, где первый элемент - название таблицы, а второй (при where=True) - условие WHERE.
- `where` (bool) - если True, ожидается условие WHERE в parameters.
- `values` (list) - значения для подстановки в условие (если используются плейсхолдеры ~).

### `delete(parameters: list, where: bool = False, values: list = None, auto_commit: bool = False)`
Удаляет записи из таблицы.
- `parameters` (list) - список, где первый элемент - название таблицы, а второй (при where=True) - условие WHERE.
- `where` (bool) - если True, удаляет только записи, соответствующие условию.
- `values` (list) - значения для подстановки в условие.
- `auto_commit` (bool) - автосохранение изменений.

### `update(parameters: list, values: list = None, where: bool = False, auto_commit: bool = False)`
Обновляет записи в таблице.
- `parameters` (list) - список из 3-4 элементов:
  - название таблицы
  - столбцы для обновления (можно с плейсхолдерами ~)
  - новые значения
  - условие WHERE (если where=True)
- `values` (list) - значения для подстановки в плейсхолдеры.
- `where` (bool) - если True, ожидается условие WHERE в parameters.
- `auto_commit` (bool) - автосохранение изменений.

### `request(request_text: str)`
Выполняет произвольный SQL-запрос.
- `request_text` (str) - SQL-команда для выполнения.

### `commit()`
Сохраняет изменения в БД (используется при auto_commit=False в других методах).

### `close()`
Закрывает соединение с БД.

# Пример использования
```
import gosqllite3

db = gosql()
db.connect("test.db")

# Создание таблицы users
db.create(["users", "id INTEGER PRIMARY KEY, name TEXT, age INTEGER"])

# Вставка данных
db.insert(["fd", "name = ~,age = ~"], ["Alice", 25], auto_commit=True)

# Выборка данных
all_users = db.selectall(["users"])
adults = db.selectall(["users", "age > 18"], where=True)

# Обновление данных
db.update(["users", "age", "30", "name = 'Alice'"], where=True)

# Удаление данных
db.delete(["users", "age < 18"], where=True)

db.close()```
