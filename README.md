# Документация gosqllite3 (v0.3)

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
- `values` (list) - значения для подстановки в условие. (можно с плейсхолдерами ~)
- `auto_commit` (bool) - автосохранение изменений.

### `update(parameters: list, values: list = None, where: bool = False, auto_commit: bool = False)`
Обновляет записи в таблице.
- `parameters` (list) - список из 3-4 элементов:
  - название таблицы
  - столбцы для (можно с плейсхолдерами ~)
  - новые значения (можно с плейсхолдерами ~)
  - условие WHERE (если where=True) (можно с плейсхолдерами ~)
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
`import gosqllite3`

# Инициализация и подключение
`db = gosqllite3.gosql()`
`db.connect("example.db")`

# 1. СОЗДАНИЕ ТАБЛИЦ
# Простая таблица пользователей
`db.create(["users", "id INTEGER PRIMARY KEY, name TEXT, age INTEGER, status TEXT"])`

# Таблица заказов с внешним ключом
`db.create(["orders", """
    order_id INTEGER PRIMARY KEY,
    user_id INTEGER,
    product TEXT,
    price REAL,
    date TEXT,
    FOREIGN KEY(user_id) REFERENCES users(id)
"""])`

# 2. ВСТАВКА ДАННЫХ
# Простая вставка
`db.insert(["users", "name, age, status"], ["Иван", 30, "active"], auto_commit=True)`

# Вставка с плейсхолдерами
`db.insert(["users", "name = ~, age = ~, status = ~"], 
          ["Мария", 25, "premium"], 
          auto_commit=True)`

# Множественная вставка (в цикле)
`users_data = [
    ["Алексей", 35, "active"],
    ["Ольга", 28, "inactive"],
    ["Дмитрий", 40, "premium"]
]`
`for user in users_data:
    db.insert(["users", "name, age, status"], user)
db.commit()  # Фиксируем все изменения`

# 3. ВЫБОРКА ДАННЫХ
# Получить всех пользователей
`all_users = db.selectall(["users"])
print("Все пользователи:", all_users)`

# Выборка с условием
`active_users = db.selectall(["users", "status = 'active'"], where=True)
print("Активные пользователи:", active_users)`

# Выборка с сортировкой (через request)
`sorted_users = db.request("SELECT * FROM users ORDER BY age DESC")
print("Пользователи по возрасту:", sorted_users)`

# 4. ОБНОВЛЕНИЕ ДАННЫХ
# Простое обновление
`db.update(["users", "status", "'inactive'", "age > 35"], where=True)`

# Обновление с плейсхолдерами
`db.update(["users", "age = ~, status = ~", "", "name = ~"],
          values=[26, "active", "Мария"],
          where=True)`

# Обновление с вычислениями
`db.update(["users", "age", "age + 1"])  # Всем +1 год`

# 5. УДАЛЕНИЕ ДАННЫХ
# Удаление по условию
`db.delete(["users", "status = 'inactive'"], where=True)`

# Очистка таблицы (опасно!)
`db.delete(["users"])`

# 6. РАБОТА С СВЯЗАННЫМИ ТАБЛИЦАМИ
# Добавляем заказы
`db.insert(["orders", "user_id, product, price, date"],
          [1, "Ноутбук", 999.99, "2023-10-01"])`

# Получаем заказы конкретного пользователя
`user_orders = db.selectall(["orders", "user_id = 1"], where=True)`

# 7. ТРАНЗАКЦИИ
`try:
    # Начало транзакции (auto_commit=False)
    db.insert(["users", "name, age"], ["Транзакция", 99], auto_commit=False)
    db.update(["users", "status", "'test'", "name = 'Транзакция'"], 
              where=True, auto_commit=False)
    db.commit()  # Подтверждаем изменения`
`except:
    db.conn.rollback()  # Откатываем при ошибке`

# 8. АГРЕГАТНЫЕ ФУНКЦИИ 
# Используем request для сложных запросов
`stats = db.request("""
    SELECT 
        COUNT(*) as total_users,
        AVG(age) as avg_age,
        MAX(age) as max_age
    FROM users
""")`
`print("Статистика:", stats)`

# Закрытие соединения
`db.close()`

# 9. ДОПОЛНИТЕЛЬНЫЕ ПРИМЕРЫ 
# Работа с датами
`db.request("INSERT INTO orders VALUES (NULL, 1, 'Телефон', 599.99, date('now'))")`

# Обновление с подзапросом
`db.request("""
    UPDATE users 
    SET status = 'gold' 
    WHERE id IN (SELECT user_id FROM orders WHERE price > 500)
""")`

# Удаление устаревших данных
`db.delete(["orders", "date < date('now', '-30 days')"], where=True)`
